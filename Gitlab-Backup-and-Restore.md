
# GitLab Backup and Restore Guide

## To Backup GitLab

### 1. Edit the `/home/ubuntu/data/gitlab/config/gitlab.rb` file and add the following:

```ruby
### Backup Settings
###! Documentation: https://docs.gitlab.com/omnibus/settings/backups.html

###! Backup and restore documentation: https://docs.gitlab.com/ce/raketasks/backup_restore.html#backup-archive-permissions
gitlab_rails['backup_archive_permissions'] = 0644
gitlab_rails['backup_pg_schema'] = 'public'

###! Set the backup retention period (in seconds) before old backups are deleted
gitlab_rails['backup_keep_time'] = 604800

gitlab_rails['backup_upload_connection'] = {
   'provider' => 'AWS',
   'region' => 'ap-southeast-1',
   'aws_access_key_id' => '<your_access_key_id>',
   'aws_secret_access_key' => '<your_secret_access_key>'
}
gitlab_rails['backup_upload_remote_directory'] = 'stonecodelabs-gitlab-backup'
gitlab_rails['backup_multipart_chunk_size'] = 104857600

gitaly['env'] = {
    'AWS_ACCESS_KEY_ID' => '<your_access_key_id>',
    'AWS_SECRET_ACCESS_KEY' => '<your_secret_access_key>'
}

gitaly['configuration'] = {
    backup: {
        go_cloud_url: 's3://stonecodelabs-gitlab-backup?region=ap-southeast-1'
    }
}
```

### 2. Refresh the GitLab configuration:
```bash
$ sudo docker exec -t gitlab gitlab-ctl reconfigure
```

### 3. To create a backup, run the following command (it can also be scheduled via a Nomad Batch Job or similar):
```bash
$ sudo docker exec -t gitlab gitlab-backup create STRATEGY=copy BACKUP=prod REPOSITORIES_SERVER_SIDE=true
```

## To Restore GitLab

### 1. Create a new directory and sub-directories for GitLab:
```bash
# Example
$ cd /home/ubuntu
$ mkdir -p docker/gitlab_backup/{data,logs,registry}  # Directory names can be customized
```

### 2. Copy the production config/ folder (which has been backed up separately) to `/home/ubuntu/docker/gitlab_backup/`. Then unzip it.
```bash
$ aws s3 cp s3://stonecodelabs-infra-backup/latitude-prod/config_backup.zip /home/ubuntu/docker/gitlab_backup/
$ unzip config_backup.zip
# Enter zip password

# Move the config folder to /home/ubuntu/docker/gitlab_backup/
$ mv home/ubuntu/data/gitlab/config/ .

# Clean up
$ rm -rf config_backup.zip home
```

### 3. If necessary, update the contents of `/home/ubuntu/docker/gitlab/config/gitlab.rb`, especially the IPs, credentials, ports, and any other relevant configurations.

### 4. Create a Docker Compose file:
```bash
$ cd /home/ubuntu/docker/gitlab_backup
$ vim docker-compose.yml
```

Then, paste the following:
```yaml
version: '3.8'
services:
  gitlab_backup:  # This name can be customized to avoid conflicts with the original service
    user: root
    image: gitlab/gitlab-ee:17.4.2-ee.0
    container_name: gitlab_backup
    restart: always
    privileged: true  # To ensure Docker-in-Docker (dind) functionality if required
    hostname: '<host ip>'
    environment:
      GITLAB_OMNIBUS_CONFIG: |  # Ports have been adjusted to avoid conflicts, but feel free to modify them as needed
        external_url 'http://<host ip>:9929/'
        gitlab_rails['gitlab_shell_ssh_port'] = 3424
        # Enable the Container Registry
        registry_external_url 'http://<host ip>:6005/'
    ports:
      - '<host ip>:9929:9929'
      - '<host ip>:9930:9930'
      - '<host ip>:3424:22'
      - '6005:6005'  # Port for Container Registry
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
      - './registry:/var/opt/gitlab/gitlab-rails/shared/registry'  # Storage for the Container Registry
    shm_size: '256m'
    networks:
      - gitlab-network-backup

networks:
  gitlab-network-backup:  # The network name has also been changed
    driver: bridge
```

### 5. Start the `gitlab_backup` container:
```bash
$ cd /home/ubuntu/docker/gitlab_backup
$ sudo docker compose up -d
# To view logs from the running GitLab container:
$ sudo docker logs -f gitlab_backup
```

### 6. After copying the config/ directory as instructed in step #2, refresh the GitLab configuration with the following command:
```bash
$ sudo docker exec -t gitlab_backup gitlab-ctl reconfigure
```

### 7. Download the backup file from AWS S3 and place it in `/home/ubuntu/gitlab_backup/data/backups`:
```bash
$ aws s3 cp s3://stonecodelabs-gitlab-backup/prod_gitlab_backup.tar /home/ubuntu/docker/gitlab_backup/tmp/
$ sudo mv /home/ubuntu/docker/gitlab_backup/tmp/prod_gitlab_backup.tar /home/ubuntu/docker/gitlab_backup/data/backups/
```

### 8. Access the `gitlab_backup` container to proceed with the restoration process:
```bash
$ sudo docker exec -it gitlab_backup bash
$ whoami  # Ensure you are logged in as root
$ chown git:git /var/opt/gitlab/backups/prod_gitlab_backup.tar
$ gitlab-backup restore BACKUP=prod

# Confirm the restoration when prompted to replace the existing tables.
# Type 'yes' since this gitlab_backup container is empty.
```
