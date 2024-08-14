---
title: "Backup Proxmox VE to the CLOUD! Backup Hook Scripts and S3"
datePublished: Wed Aug 14 2024 03:47:59 GMT+0000 (Coordinated Universal Time)
cuid: clztb9s99000v09la16fqdurm
slug: backup-proxmox-ve-to-the-cloud-backup-hook-scripts-and-s3
canonical: https://www.apalrd.net/posts/2022/pve_backup/
tags: virtualization, backup, pve, proxmox

---

Proxmox has a pretty good backup scheduler, but it relies on the backup destination being mounted as a storage location. This implies that the backup destination needs to be a protocol that Proxmox supports - SMB (CIFS), NFS, … or Proxmox Backup Server. If you want to push your backups to a cloud service, you probably need something a bit more complicated. Thankfully, Proxmox’s backup scheduler thought about this and has a hook feature we can use for this purpose, and we can use any protocol supported on the Debian base system, including things such as FUSE or s3cmd.

In this project, I integrate S3 object storage with the Proxmox backup scheduler, copying resulting backups to a cloud storage service directly from the Proxmox system. The S3 protocol is ubiquitous among cloud object storage providers, so we aren’t tied to AWS - we can use any service such as Backblaze. In my case, I’m demonstrating this with Linode.

# Proxmox VZDump hook scripting

Proxmox’s documentation mentions briefly that the `--script` argument can be passed to `vzdump`, the Proxmox backup utility. We can also add the line `script <path_to_script>` in the `/etc/pve/jobs.conf` file, and it will retain this script argument even if you modify the backup job from the GUI. You can’t add a script command from the GUI, but you need to initially create the job there.

P[roxm](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)ox mentions a backup file ([`vzdump-hook-script.pl`](http://vzdump-hook-script.pl)), but I was unable to find a copy easily. For your convenience, I’ve uploaded it to my site [here](https://www.apalrd.net/posts/2022/pve_backup/vzdump-hook-script.pl) for you to view. The tl;dr is that:

* The first argument to the script is the phase, which is one of:
    
    * job-start
        
    * job-end
        
    * job-abort
        
    * backup-start
        
    * backup-end
        
    * backup-abort
        
    * log-end
        
    * pre-stop
        
    * pre-restart
        
    * post-restart
        
* The ‘job’ type phase[s ar](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)e called once for a job, while the rest are called for each backup in the job
    
* The backup mode (stop/suspend/snapshot) and vmid are passed as arguments to the non-job types
    
* Addition[al i](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)nformation is passed as environment variables for certain phase[s, i](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)ncluding [`TARG`](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)`ET` (the [ful](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)l path on [the](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) local syste[m to](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) the backu[p fi](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)le), `LOGFILE` [(th](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)e full [path](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) to the [log](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) file), and [a fe](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)w other useful [att](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)ributes.
    

In our case, we are going to hook to the `job-end`, `backup-end`, and `log-end` phases and do t[he f](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)ollowing:

* For `backup-end`, copy the file `TARGET` to the S3 bucket and delete the original
    
* [For](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) `log-end`, copy the file `LOGFILE` to the S3 bucket and delete the original
    
* For `job-end`, check if there are any backups which should be expired and remove them. This is done only once, even if there are multiple VMs to backup [in a](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) single job.
    

# S3 Tools

There are a few different open source options for general purpose file manag[emen](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)t using the S3 protocol. The most common are `s3cmd` and `s3fs-fuse`. The first p[rovi](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)des a command line utility to perform actions on an S3 bucket (ls, put, get[, de](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)lete, …) and the second implements a filesystem backed by an S3 bucket. Since the S3 protocol can only replace files (not modify parts of a file), `s3fs-fuse` can be qu[ite](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) slow d[epen](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)ding on how files are written/read. Given the use of hook scripts here, I’ve decided to use `s3cmd`. It’s easily installed by running `apt install s3cmd` on Proxmox.

Once `s3cmd` is installed, we need to configure it by running `s3cmd --configure`. It will ask you a few questions, and the answers will depend on your cloud storage provider. They should have a guide on configuring S3 access with their service (i.e. Linode’s is [here](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/)). I’ve walked through my own setup in the video.

Once we run configure, it will generate a file called `~/.s3cfg` which includes the access key and endpo[int](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd/) URL. Since we ran this as root, it will be `/root/.s3cfg` on our Proxmox host. That’s fine, since backup jobs run as root.

If you’re interested in the details, feel free to read the [s3cmd docs here.](https://s3tools.org/usage)

Once you’ve verified that you have `s3cmd` installed and can `s3cmd ls` and `s3cmd put` to the target bucket, we can add the s3 put’s to our backup script[.](https://gist.github.com/JProffitt71/9044744)

# Backup Expiration Script

I [found a script](https://gist.github.com/JProffitt71/9044744) to handle expiration of files in S3. I’ve copied it here for reference. I named it `s3cleanup.sh` on my system.

```bash
#!/bin/bash

# Usage: ./s3cleanup.sh "bucketname" "7 days"

s3cmd ls s3://$1 | grep " DIR " -v | while read -r line;
  do
    createDate=`echo $line|awk {'print $1" "$2'}`
    createDate=$(date -d "$createDate" "+%s")
    olderThan=$(date -d "$2 ago" "+%s")
    if [[ $createDate -le $olderThan ]];
      then
        fileName=`echo $line|awk {'print $4'}`
        if [ $fileName != "" ]
          then
            printf 'Deleting "%s"\n' $fileName
            s3cmd del "$fileName"
        fi
    fi
  done;
```

Don’t forget to `chmod +x` it once you are done.

# Final Backup Script

Here is the final backup script which you can copy to your system. Again, don’t forget to `chmod +x` it.

```perl
#!/usr/bin/perl -w
# Hook script for vzdump to backup to an S3 bucket
# 2022 Andrew Palardy
# Based on example hook script from Proxmox

use strict;

# Define the name of your bucket here
my $bucket = "apalrd-proxmox";
# The S3 endpoint comes from the s3cmd --configure setup, it is not set here

# Define the number of days to retain backups in the bucket
# Only accepts days, doesn't check hours/minutes
my $retain = 30;


#Uncomment this to see the hook script with arguments (not required)
#print "HOOK: " . join (' ', @ARGV) . "\n";

#Get the phase from the first argument
my $phase = shift;

#For job-based phases
#Note that job-init was added in PVE 7.2 or 7.3 AFAIK
if (    $phase eq 'job-init'  ||
        $phase eq 'job-start' || 
        $phase eq 'job-end'   || 
        $phase eq 'job-abort') { 

        #Env variables available for job based arguments
        my $dumpdir = $ENV{DUMPDIR};

        my $storeid = $ENV{STOREID};

        #Uncomment this to print the environment variables for debugging
        #print "HOOK-ENV: dumpdir=$dumpdir;storeid=$storeid\n";

        #Call s3cleanup at job end
        if ($phase eq 'job-end') {
                system ("/root/s3cleanup.sh $bucket \"$retain days\"");
        }
#For backup-based phases
} elsif ($phase eq 'backup-start' || 
         $phase eq 'backup-end' ||
         $phase eq 'backup-abort' || 
         $phase eq 'log-end' || 
         $phase eq 'pre-stop' ||
         $phase eq 'pre-restart' ||
         $phase eq 'post-restart') {

        #Data available to backup-based phases
        my $mode = shift; # stop/suspend/snapshot

        my $vmid = shift;

        my $vmtype = $ENV{VMTYPE}; # lxc/qemu

        my $dumpdir = $ENV{DUMPDIR};

        my $storeid = $ENV{STOREID};

        my $hostname = $ENV{HOSTNAME};

        my $tarfile = $ENV{TARGET};

        my $logfile = $ENV{LOGFILE}; 

        #Uncomment this to print environment variables
        #print "HOOK-ENV: vmtype=$vmtype;dumpdir=$dumpdir;storeid=$storeid;hostname=$hostname;tarfile=$tarfile;logfile=$logfile\n";

        # During backup-end, copy the target file to S3 and delete the original on the system
        if ($phase eq 'backup-end') {
                #S3 put
                my $result = system ("s3cmd put $tarfile s3://$bucket/");
                #rm original
                system ("rm $tarfile");
                #Die of error returned
                if($result != 0) {
                        die "upload backup failed";
                }
        }

        # During log-end, copy the log file to S3 and delete the original on the system (same as target file)
        if ($phase eq 'log-end') {
                my $result = system ("s3cmd put $logfile s3://$bucket/");
                system ("rm $logfile");
                if($result != 0) {
                        die "upload logfile failed";
                }
        }
#Otherwise, phase is unknown
} else {

        die "got unknown phase '$phase'";

}

exit (0);
```

src :

* [https://www.apalrd.net/posts/2022/pve\_backup/](https://www.apalrd.net/posts/2022/pve_backup/)
    

* [https://github.com/andrizan/proxmox-ve-backup-cloud](https://github.com/andrizan/proxmox-ve-backup-cloud)