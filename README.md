# Restic template

This is a [Restic](https://restic.net/) template I use for most of my computers. It's mainly meant for me to clone when setting up a new computer, but there is nothing stopping anyone else from using it as well. The file encrypted `cmd.sh.gpg` contains the same type of command as the `restic backup` example below, but with the hostnames and folders that are specific for my setup, and is only meant to be used by me.

## Server setup
Have a linux server running with SSH access enabled. [Install restic](https://restic.readthedocs.io) on it. Initiate a new restic repo somewhere on it:

```bash
restic init --repo /srv/restic-repo
```

Done, the rest of the work is done by the client. See section below about pruning snapshots though, as it is probably something you want to do.

## Client setup
Do these steps on the computer you want to backup.

Decrypt the `cmd.sh.gpg` file using if you are me,

```bash
gpg -d cmd.sh.gpg > cmd.sh
```

or create your own based on the example below.

```bash
restic backup \
    --repo sftp:user@host:/path/to/restic_repo/ \
    --password-file /home/user/.restic_password \
    --exclude-file /home/user/restic_template/exclude.txt \
    --one-file-system \
    --exclude-caches \
    --verbose \
    /home/ \
    /etc/ \
    /root/ \
    /var/lib/docker/volumes
```

Then create a new file containing the same password as for the restic repo you just created on the server:

```bash
touch ~/.restic_password ; chmod 600 ~/.restic_password
echo -n mysecretpassword > ~/.restic_password

# remove your super secret password from the command history
history -d $((HISTCMD-2))
```

Run `cmd.sh` as a cronjob daily. Make sure you have generated ssh keys and copied the public key to the remote server for [passwordless login](https://www.google.com/search?q=generate%20ssh%20keys%20passwordless%20login).

```bash
# in short
ssh-keygen
ssh-copy-id user@host.com
```


## Mounting the repo and restoring files
You can mount the repo and browse it as a file structure you can copy files from. On the server:

```bash
# create an empty directory to be used as mount point
mkdir mnt

# mount the repo
restic mount -r /path/to/repo/ mnt
```

Restic will ask you for your password and then mount the repo. The terminal will be busy keeping the mount alive, and when you are finished you can press `ctrl+c` to unmount it.

If you prefer not to open a new ssh connection to your server just to browser the files, you can send the mount process to the background by pressing `ctrl+z`, and then let the process continue in the background by typing `bg` This will let you browse the files in your terminal. When you are done, simply bring the mount process to the foreground by typing `fg` and then close it down with `ctrl+c` as usual.



## Additional things

### Run as root user on client
If you want to backup places like docker volumes etc, you will have to run `cmd.sh` as the root user as it is the only one who has access to all files. Create the `.restic_password` file in the root user's home folder, setup ssh keys for the root user, and run the cronjob in the root users crontab instead.

### Prune old snapshots
Set up a cronjob on the server as the user you login in with when running `cmd.sh` that will prune old snapshots. Create a password file just like in the client setup and create and modify the cronjobs below.

```bash
0 9 * * *  restic check --repo /path/to/restic_repo/ --password-file /home/user/.restic_password
0 10 * * * restic forget --prune --repo /path/to/restic_repo/ --password-file /home/user/.restic_password --keep-hourly 24 --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --keep-yearly 7
```

