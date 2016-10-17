# rcloned
Bash scripts for near real-time mirroring of your data to the Cloud using rclone

# Description
I'm not much of a programmer but this works for my needs as all I have is a small office. I hacked some Bash code together with pretty much toothpaste and duct tape and did not comment it that well at all.

As a point of reference my data on my linux box (running CentOS 7) is in /dataEncrypted/Folder 1/ ; /dataEncrypted/Folder 2/ ; etc. I use EncFs for the encryption/decryption and the decrypted folder is /data. And my Cloud data (in rclone format) is in acd:/Folder 1/ ; acd:/Folder 2/ ; etc.

You'll need to install inotifywait (for Centos 7 it's inotify-tools from the EPEL repo) and perl and perhaps some other dependencies and modify the scripts to fit your data location and rclone config.  I then placed all the scripts in /etc/local/sbin/ and then chmod +x.

Then, its as simple as "rcloned start", "rcloned status", "rcloned stop".  The logfile is /var/log/rcloned.log.

There are two Bash scripts that do all the work: rclonedWatcher looks for any writes to any file within a folder, recursively, and puts those filenames in a pipe: /tmp/rcloned. rclonedProcessor looks for and filenames in that pipe, every 15 seconds, and uploads all the files in the pipe, one by one, to the Cloud.

I use 2 cron jobs. The first one runs every minute and thereby starts rclonedWatcher and rclonedProcessor and checks to ensure all processes are running, and will restart all processess if any processs crashes. The last one runs every night at midnight to fix anything missed by the rclonedWatcher and rclonedProcessor. And since I only have rclonedWatcher looking for writes (and nothing else), the last cron job also takes care of file deletions, filename changes, file moves, etc.

```
* * * * * /usr/local/sbin/rclonedUptime

0 0 * * * folders=("/Folder 1/" "/Folder 2/"); for i in "${folders[@]}"; do [ -d "/dataEncrypted$i" ] && (pgrep -f "rclone sync" || /usr/local/sbin/rclone sync --bwlimit=1M "/dataEncrypted$i" "acd:$i"); done;
```

# Mirroring (Not A Backup)
Remember that this is NOT a backup program.  It simply mirrors your local data to the Cloud.  You still need backups.  I recommend taking snapshots of your data (and storing them on AT LEAST one different physical device as your main data) so you can go back in time and recover your files as they existed in the past.  Why?  Imagine you're working on your 100-page thesis which took you 6 months to create and which gets corrupted while saving locally for whatever reason, that corrupted file is then mirrored by rcloned to your Cloud.  Congratulations, you now you have two perfect copies of a corrupt file.
