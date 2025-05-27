# Nextcloud with OnlyOffice and Let's Encrypt Using Docker Compose

I have forked heyValdemar's project and corrected for my needs.
Place project files to folder, which will contain all project files (databases, nextcloud data, etc)

❗ Change variables in the `.env` to meet your requirements.

💡 Note that the `.env` file should be in the same directory as `nextcloud-traefik-letsencrypt-docker-compose.yml`.

Deploy Nextcloud using Docker Compose:

```
docker compose -f nextcloud-onlyoffice-traefik-letsencrypt-docker-compose.yml -p nextcloud up -d
```
## Background Jobs Using Cron

To ensure your Nextcloud instance operates efficiently, it's important to use the "Cron" method to execute background jobs. A dedicated Docker container has already been set up in your environment to handle these tasks.

### Steps to Enable Cron:

1. **Log in to Nextcloud as an Administrator.**
2. Go to **Administration settings** (click on your user profile in the top right corner and select "Administration settings").
3. In the **Administration** section on the left sidebar, select **Basic settings**.
4. Scroll down to the **Background jobs** section.
5. Select the **"Cron (Recommended)"** option.

![nextcloud-cron](https://github.com/user-attachments/assets/bf3399ef-c859-499b-b992-fd5d4eeb5f0b)

### Why Use Cron?

The "Cron" method ensures that background tasks, such as file indexing, notifications, and cleanup operations, run at regular intervals independently of user activity. This method is more reliable and efficient than AJAX or Webcron, particularly for larger or more active instances, as it does not depend on users accessing the site to trigger these tasks. With the dedicated container in your setup, this method keeps your Nextcloud instance responsive and in good health by running these jobs consistently.

## Backups

I am using full VM's backup and recommend using it.

## Disabling Skeleton Directory for New Users

New Nextcloud users typically receive default files and folders upon account creation, which are sourced from the skeleton directory. Disabling this feature can be useful to provide a clean start for users and reduce disk usage. Use the `occ config:system:set` command to set the skeleton directory path to an empty string, effectively disabling the default content for new users.

Run the command below.
```
docker exec -u www-data -it nextcloud_app php occ config:system:set skeletondirectory --value=''
```
## Fixing Database Index Issues

Your Nextcloud database might be missing some indexes. This situation can occur because adding indexes to large tables can take considerable time, so they are not added automatically. Running `occ db:add-missing-indices` manually allows these indexes to be added while the instance continues running. Adding these indexes can significantly speed up queries on tables like `filecache` and `systemtag_object_mapping`, which might be missing indexes such as `fs_storage_path_prefix` and `systag_by_objectid`.

Run the command below.
```
docker exec -u www-data -it nextcloud_app php occ db:add-missing-indices
```
Confirm the indices were added by checking the status:
```
docker exec -u www-data -it nextcloud_app php occ status
```
- Operations on large databases can take time; consider scheduling during low-usage periods.
- Always backup your database before making changes.

## Rescanning Files

When files are added directly to Nextcloud's data directory through methods other than the web interface or sync clients (e.g., via FTP or direct server access), they are not automatically visible in the Nextcloud user interface. This happens because these files bypass Nextcloud's normal indexing process.

To make all manually added files visible in the UI, you can use the `occ files:scan` command to update Nextcloud's file index. This command should be used with care as it can impact server performance, especially on larger installations.

Run the command below.
```
docker exec -u www-data -it nextcloud_app php occ files:scan --all
```
- Be aware that this command can significantly affect performance during its execution. It is advisable to run this scan during periods of low user activity.
- Always ensure that you have up-to-date backups before performing any operations that affect the filesystem or database.

## Configure Nextcloud with OnlyOffice. 
Go inside the container :
```
docker exec -u www-data -ti nextcloud_app bash
```
allow to set onlyoffice as local container. Within the nextcloud container :
```
php occ --no-warnings config:system:set allow_local_remote_servers --value=true
```
add hostame container and domain to trusted domain :

```
php occ --no-warnings config:system:set trusted_domains 1 --value="onlyoffice"
php occ --no-warnings config:system:set trusted_domains 2 --value="nextcloud"
php occ --no-warnings config:system:set trusted_domains 3 --value="<YOUR NEXTCLOUD_DOMAIN>"
php occ --no-warnings config:system:set trusted_domains 4 --value="<YOUR ONLYOFFICE DOMAIN>"
```

go to your nextlcoud instance, install it, go to app and add Onlyoffice from the app store. Configure it

```
php occ --no-warnings config:system:set onlyoffice DocumentServerUrl --value="http://<ONLYOFFICE.DOMAIN>"
php occ --no-warnings config:system:set onlyoffice DocumentServerInternalUrl --value="http://onlyoffice/"
php occ --no-warnings config:system:set onlyoffice StorageUrl --value="http://nextcloud/"
php occ --no-warnings config:system:set onlyoffice jwt_secret --value="<JWT SECRET FROM .env FILE>"
php occ --no-warnings config:system:get onlyoffice
php occ onlyoffice:documentserver --check
```
