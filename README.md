# Restic template

### init
Decrypt the cmd.sh file using

```bash
$ gpg -d cmd.sh.gpg
```

and edit the command to include the locations you want backed up. Then create a new file containing the restic repo password:

```bash
$ touch restic_password ; chmod 600 restic_password
$ echo -n mysecretpassword > restic_password
```

### usage
Run the backup command from cmd.sh as a cronjob of the root user daily.



