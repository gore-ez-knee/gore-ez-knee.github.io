---
title: Backup/Restore GitLab
date: 2022-06-04 10:00:00 -0500
categories: [Misc, GitLab]
tags: [gitlab, aws, s3, docker, docker-compose, gitlab-ctl]
image:
  src: /assets/img/post_images/gitlab_backup_restore/gitlab-banner.jpg
  width: 1920
  height: 300
---

## Intro

Do you have a small GitLab container running on an instance in AWS for yourself or a small team? Have you ever needed to back-up GitLab and move it to another machine wether that be on another instance or on-prem? The following are instructions on how to back-up a GitLab container to an AWS S3 bucket, deploy GitLab to another environment, pull down the backup, and finally restore the back-up on the new machine. 

The instructions are assuming you have GitLab deployed on a container, but it should still work if you manually installed GitLab on a machine. I would recommend at least having it on a container, because it's way easier to update. My example Docker Compose file in Step 6 has it setup in a way that always keeps GitLab updated.  

## Step 1. Create S3 Bucket
Create an S3 bucket. The [architecture diagram](https://docs.gitlab.com/ee/install/aws/manual_install_aws.html#architecture) shows you can create multiple buckets for different GitLab objects. In this case, we're just worried about setting up a backup bucket.
## Step 2. Create an IAM Policy for the Instance that is housing GitLab.
Using [GitLab's Documentation](https://docs.gitlab.com/ee/install/aws/manual_install_aws.html#create-an-iam-policy), create an EC2 instance role for GitLab using the provided example policy. If all you are worried about is backups, just plug the backup bucket ARN into the resource.
Example Policy:
```
{   "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::gl-backup-example-bucket/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:AbortMultipartUpload",
                "s3:ListMultipartUploadParts",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": "arn:aws:s3:::gl-backup-example-bucket"
        }
    ]
}
```
Once the policy has been created, attach the EC2 instance role to the instance.
## Step 3. Modify `gitlab.rb` to upload backups to S3
- Drop into the the container (if you are using Docker): `sudo docker exec -it <container-name/id> /bin/bash`
- Modify the `gitlab.rb` file: `vi /etc/gitlab/gitlab.rb`
- Add the following to the `gitlab.rb` file:
    ```
    gitlab_rails['backup_upload_connection'] = {
    'provider' => 'AWS',
    'region' => 'us-gov-west-1',
    'use_iam_profile' => true
    }
    gitlab_rails['backup_upload_remote_directory'] = 'gl-backup-example-bucket'
    ```
- Once added, reconfigure GitLab: `gitlab-ctl reconfigure`
## Step 4. Create a Backup
- Once the reconfigure has completed, you can create a backup from within the container:
    ```
    gitlab-backup create SKIP=registry
    ```
- OR you can run the command from Docker:
    ```
    sudo docker exec -it gitlab gitlab-backup create SKIP=registry
    ```

> Note: I skip the registry to avoid an error I received when trying to restore from backup. It got mad at me for not having a registry.gz file, or something like that. Since I wasn't using GitLabs registry, I just skipped it hence the `SKIP=registry`
{: .prompt-info }

- Once the command finishes, check the S3 bucket for a backup file. This command will also create a backup file in `/var/opt/gitlab/backups/`
## Step 5. Manually Backup the `gitlab-secrets.json` and `gitlab.rb` Files
When you create a backup, it does not backup the `gitlab-secrets.json` or `gitlab.rb` files. It even tells you you must back those two files up manually. I found that it was easier to push those files to the same bucket as the backups and pull them to the new instance.
In our case, using Docker...
- Copy the files from the mounted Docker volumes: `sudo cp /srv/gitlab2/config/gitlab.rb /srv/gitlab2/config/gitlab-secrets.json .`
- Move those files to your bucket:
    ```
    aws s3 cp gitlab.rb s3://gl-backup-example-bucket/
    aws s3 cp gitlab-secrets.json s3://gl-backup-example-bucket/
    ```
## Step 6. Restore from a Backup
Whether you need to restore from a failure or move your GitLab to another instance, here is where you will restore the backup.
ALSO the following commands assume you are running GitLab in a container. So on your new server: install docker, start docker, and install docker-compose. Next create a `docker-compose.yaml` file. Here is a sample one I use:
```
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    container_name: gitlab
    restart: always
    hostname: 'example.gitlab.com'
    ports:
      - '80:80'
      - '443:443'
      - '6222:22'
    volumes:
      - '/srv/gitlab/config:/etc/gitlab'
      - '/srv/gitlab/logs:/var/log/gitlab'
      - '/srv/gitlab/data:/var/opt/gitlab'
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 300 --cleanup
```
Watchtower is here to check for new GitLab images every 5 minutes and deploys them with the same configs as well as clean up old images.
- Spin up the containers: `sudo docker-compose up -d`
- Wait for the GitLab container to be healthy: `sudo watch docker ps`
- Once healthy, pull down the `gitlab.rb` file and place it into the container, chown it to root, and reconfigure:
    ```
    aws s3 cp s3://gl-backup-example-bucket/gitlab.rb .
    sudo docker cp gitlab.rb gitlab:/etc/gitlab/
    sudo docker exec -it gitlab chown root: /etc/gitlab/gitlab.rb
    sudo docker exec -it gitlab gitlab-ctl reconfigure
    ```
- Once the reconfigure is finished, pull down the latest backup, place it into the container and chown it to git:
    ```
    aws s3 cp s3://gl-backup-example-bucket/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar .
    sudo docker cp 11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar gitlab:/var/opt/gitlab/backups/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar
    sudo docker exec -it gitlab chown git:git /var/opt/gitlab/backups/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar
- Stop the puma and sidekiq services and confirm the services are down:
    ```
    sudo docker exec -it gitlab gitlab-ctl stop puma
    sudo docker exec -it gitlab gitlab-ctl stop sidekiq
    sudo docker exec -it gitlab gitlab-ctl status
    ```
- Restore the backup. Remember to ignore the `_gitlab_backup.tar`:
    ```
    sudo docker exec -it gitlab gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce
    ```
## Step 7. Load the `gitlab-secrets.json` file and Restart
- Copy the file into the container, chown it root, and reconfigure:
    ```
    aws s3 cp s3://gl-backup-example-bucket/gitlab-secrets.json .
    sudo docker cp gitlab-secrets.json gitlab:/etc/gitlab/
    sudo docker exec -it gitlab chown root: /etc/gitlab/gitlab-secrets.json
    sudo docker exec -it gitlab gitlab-ctl reconfigure
    ````
- Restart GitLab:
    ```
    sudo docker exec -it gitlab gitlab-ctl restart
    ```
- And for safe measure, just restart the container:
    ```
    sudo docker restart gitlab
    ```
The `gitlab-ctl restart` might be irrelevant with restarting the container, but that's the redundant method I ran to get this to work and I didn't double-check to see if it was necessary.
## Profit
