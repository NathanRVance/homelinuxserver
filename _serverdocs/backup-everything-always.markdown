---
title: "Back Up Everything. Always."
desc: "No data is safe, ever. But, backing it all up can make it a bit less precarious."
sortable: 5
---

So far, we've generated a fair number of configuration files, some email, and a database of Nextcloud files. These things must be backed up now, tomorrow, and every day after that. Otherwise, you run the risk of a hard disk failure destroying everything.

Luckily, routine tasks like backups are something computers are very good at. We'll set up automatic backups that operate like this:
 * On your server, all files to be backed up are rsynced to a backup folder on the server every night.
 * Next, the backup folder is rsynced to a remote machine (if available).
 * Finally, the remote machine makes a tar.gz archive of the data, places it in long term storage, and rotates stale backups out.

Let's get to work implementing this.
1. Make a shell script called `/root/backup.sh` that rsyncs all of your data to a backup location. Here's an example that backs up all of the files we have modified so far in constructing our Home Linux Server:
	```
	#!/bin/bash
	backup="/root/backups"

	# Handle rsyncs in a loop. This is a list of all folders and files we want to back up. Append additional paths to the end of the list.
	sync=(/var/www/ /etc/nginx /var/mail/ /etc/postfix/ /etc/aliases /etc/opendkim.conf /etc/default/opendkim /lib/systemd/system/opendkim.service /etc/dkimkeys/ /etc/dovecot/ /root/backup.sh)

	# Perform the rsyncs.
	for file in ${sync[@]}; do
		mkdir -p $backup/$(dirname $file)
		rsync -Aax $file $backup/$file
	done
	
	# Perform dumps (must be done individually).
	# Back up MariaDB. If you named the database something other than nextclouddb when setting up nextcloud, replace the name here.
	mysqldump --single-transaction -h localhost -u root nextclouddb > $backup/nextclouddb.bak
	
	# Back up cron.
	crontab -l > $backup/cron.bak

	remote_backup="/path/to/backups"
	remote_longterm_store="path/to/longterm"
	remote_user="username"
	remote_host="hostname"
	# Copy the backup to a remote location. The --delete flag removes extraneous destination files.
	rsync -Aaxz --delete $backup $remote_user@$remote_host:$remote_backup
	# Create a tar.gz of the backup. Replace /path/to/rotate.sh with the location you use in the next step.
	ssh $remote_user@$remote_host "cd $remote_longterm_store && tar czf $(hostname -f)-$(date +"%Y%m%d").tar.gz $remote_backup && ./rotate.sh"
	```
2. On the remote server, make the shell script `/path/to/longterm/rotate.sh`. This script is responsible for organizing your backups. The following example retains daily backups for a week, one backup from each week for a month, and one backup from each month for a year.
	```
	#!/bin/bash

	# Ensure directories exist.
	if [ ! -d week ]; then mkdir week; fi
	if [ ! -d month ]; then mkdir month; fi
	if [ ! -d year ]; then mkdir year; fi
	
	# This script is called after someone dumped a tar into this directory. Place it in the correct folder.
	mv *.tar.gz week/

	# Function returns age of a file in days
	function age {
		file=$1
		echo $((($(date +%s) - $(stat -c %Y $file)) / 86400))
	}
	
	# Function rotates backups into and out of correct folders
	function rotate {
		fromDir=$1
		moveFrom=$2
		toDir=$3
		ageGapTo=$4
		# Deal with the oldest backups in fromDir
		while :; do
			thisFile="$(echo $(ls -t $fromDir) | awk -v pos=$moveFrom '{print $pos}')"
			if [ -z "$thisFile" ]; then break; fi
			thisFile=$fromDir/$thisFile
			toYoungest=$toDir/$(ls -t $toDir/ | head -n 1)
			if (( $(age $toYoungest) - $(age $thisFile) > $ageGapTo )); then
				mv $thisFile $toDir
			else
				rm -f $thisFile
			fi
		done
	}
	
	# Syntax:
	# rotate from minIndexToMove destination minDestAgeGap(days)
	rotate week 8 month 7
	rotate month 5 year 31
	rotate year 13 . 0
	rm -f *.tar.gz
	```
3. Make both scripts executable, and then test them out. You should see that your backup directory fills with data, and that it all ends up on the remote machine.
4. Add the backup process to your root's crontab.
	```
	0 2 * * 2 /root/backup.sh
	```
