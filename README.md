# kopano-docker-restic-backup
A tiny set of scripts to automate kopano docker backups via restic to a wide range of storage

## usage
Clone this repo to your host running [kopano-docker](https://github.com/zokradonh/kopano-docker). You will need to create your own copy of *settings* and *restic.env*.
```bash
git clone https://github.com/engelant/kopano-docker-restic-backup.git
cd kopano-docker-restic-backup
cp restic.env.example restic.env
cp settings.example settings
chmod 700 restic.env settings
```
The settings file is populated with example content, you need to change the **KOPANO_DOCKER_DIR** to pint to your kopano-docker installation path.
The default path for the local backup data is this directory, but feel free to change it, if you want.

**RESTIC_RETENTION** defines the retention policy for [forgetting snapshots](https://restic.readthedocs.io/en/latest/060_forget.html) of the backups. You might want to prune the forgotten snapshots (or maybe not, see the warning in the link), but most likely you don't want to do that after every backup, as there is a lot of downloading/uploading involved, due to the way restic packs the data. 

For the remote backup you will need to edit the *restic.env* file. Check out [the restic docs](https://restic.readthedocs.io/en/latest/030_preparing_a_new_repo.html) for further information on the available environment variables.

**Keep in mind to Backup your RESTIC_PASSWORD, as your encrypted Backup is garbage without it.**

Now that everything is set up run the following command to initialize your repository and create the first backup:
```bash
./restic init
./backup-job
```

## Automate backup
You need to copy and modify the *cron.d/restic-kopano* file to */etc/cron.d/* to match the location of your *backup-job* script. The example cron will create a backup every 6 hours.
```bash
cp cron.d/restic-kopano /etc/cron.d/
```

## Check backups / interact with restic
The *./restic* command provides a wrapper for the restic binary inside the [instrumentisto/restic](https://github.com/instrumentisto/restic-docker-image) Docker image, with the environment variables set to access the backup. Just user the [restic documentation](https://restic.readthedocs.io/en/latest/index.html) for reference.

## Cloud storage
I __suggest__ the [Backblaze B2 Storage](https://www.backblaze.com/b2/cloud-storage.html), for that they have a free 10GB Tier and are pretty cheap. But also be aware to:
* Create your Account in the propper region (USA or Europe) or it will be **SLOW**
* Set your Bucket Lifecycle Settings to **Keep only the last version of the file** as restic does the lifecycle management for you

## More storage
restic supports [rclone](https://rclone.org/#providers) and therefor a huge set of storage providers. It's included in the used [instrumentisto/restic](https://github.com/instrumentisto/restic-docker-image) Docker image, but for this one you're on your own right now. Sorry.