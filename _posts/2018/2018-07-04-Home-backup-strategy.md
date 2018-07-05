---
layout: post
title: Home backup strategy
commentIssueId: 11
tags: files, backups, restic, rclone, backup strategy
---

_TL;DR:_ CrashPlan died in May 2018, so I switched my family over to using Duplicati, restic, and rclone to sync backups both locally and in the cloud. As a bonus, we're doing for free for the next year with Google Cloud Storage due to their 12 month, $300 free trial.

## Motivation

For the past two years or so from the date of this post, I've had my family using a piece of software known as CrashPlan for backing up all of the computers for my family, my siblings and their families, and my parents. CrashPlan had some faults (it tended to be a bit of a memory hog, among some other issues) but overall it performed very well for us. Code42, the creators of CrashPlan had an annual plan that could be had for $150 that allow backups of unlimited sizes for up to 10 computers. There were native clients for Windows, macOS, and Linux. It automatically kept track of snapshots and set up services that began on startup, meaning it was easy to install for relatives and forget about it. Automated reports were sent on a weekly basis giving status updates of how often backups were ran. Like I said, it worked very well for our use case of about 850 GB spread out between 10 computers. But as it almost always seems to be the case, all good things must come to an end. In the summer of 2017, it was announced that CrashPlan for Home would be discontinued in May 2018, leaving our family and other users to find a new backup home for our digital belongings.

After procrastinating for most of that period, I finally realized we needed to change backup providers a few months ago, and started thinking about a strategy that we could use to have everything safely backed up. Our digital possessions include things like our wedding photos and those of our daughter growing up, so I wanted to have a setup that would survive anything from a home invasion to a major environmental disaster. Obviously, if there's an environmental disaster, then I'm not going to be that worried about photos compared to making sure my family is safe, but if there was a system in place to take care of those things automatically while I took care of business elsewhere then all the better. 

## The Strategy

I read a number of articles and blog posts detailing how various individuals or organizations managed their backups, and it seemed that following the 3-2-1 backup strategy[^1] was a good idea. As described by BackBlaze:

> A 3-2-1 strategy means having at least 3 total copies of your data, 2 of which are local but on different mediums (read: devices), and at least 1 copy offsite. We’ll use “kitten.jpg” as an example for this scenario. Kitten.jpg lives on your computer at home, it was a picture that you took of your cat in 2012. That’s one copy of the data. You also have an external hard drive that you use for backing up your computer, if you’re on a Mac, you might be using it as a Time Machine drive (and Backblaze loves Time Machine). As part of its backup process, that external hard drive will back up kitten.jpg. That’s a second copy, on a different device or medium. In addition that external hard drive, you also have an online backup solution. The online backup continuously scans your computer and uploads your data offsite to a datacenter. Kitten.jpg is included in this upload, and that becomes the third copy of your data.

The second backup that's local allows for quick restoration and usage of the backup as needed, while the third remote backup is around in case of a major disaster of failure that could wipe out the hardware of the first and second copies, or in the event that their are errors when attempting to restore from the second backup after the first has already failed. 

## How to Backup

There are a number of consumer products out there, but none seemed to meet the intersection between cost, features, and system availability for which I was looking. 

Around this point, I came across an application called restic[^2] that was open-sourced, free, stable, and relatively efficient. I really came to like restic for a number of reasons, but thought it might have been a bit harder for my more non-technical relatives to use since it only has a command line interface. For them, I came across Duplicati[^3] which has a graphical user interface in the form of a local web server that is accessed from a browser. The biggest differences at the moment are that Duplicati does compression on the stored files, decreasing the overall size, but it maintains the state of the backup repository in a local database, which can take a very long time (sometimes weeks) to recreate.[^4] On the other hand, restic does not yet have compression (though it is in the works[^5]), and it does not rely on maintaining anything outside of the repository where the data is stored (aside from the password to decrypt a repository if it's encrypted). This is both a blessing and a curse, since you wouldn't have to recreate or repair a database on the client side, but it requires connecting to the repository to process, check, or prune the backup data. If you happen to be on a slow Internet connection (like we are) or are using a cloud service that charges for operations on your buckets/datastores (as we are), then this can lengthen the duration of the backup cycle considerably or increase the cost of your backups. Just things to consider.

## Where to Backup

Once the tools we've decided what to use to backup the files, we can then decide on where to store the backups. According to the 3-2-1 Backup Strategy, you need at least two other backups on different media, one of which should be located away from the other two copies. For the second, local backup, restic or Duplicati would backup to a second hard drive nearby, and then the third copy would be hosted on a cloud provider. Privacy issues about hosting personal files on a third party service were mitigated by the ability of restic and Duplicati to encrypt the backups locally before sending them over the wire. Next we just needed to decide on which third party to use. Depending on the program used, this can be limited to a few options, maybe even one. However, restic and Duplicati both support backups to several backends.

I initially thought of sending a server with some hard drives to the home of a relative, and installing a server to receive the backups from the client computers. CrashPlan had this implemented as a great feature, where as long as each machine had the CrashPlan client installed, you could send your backups to the other machine without any further effort required. I ended abandoning that train of thought once I realized that the amount of data we were wanting to backup wouldn't justify the cost of buying needed hardware, the effort of setting up backups in each direction, and the cost of electricity to keep both servers running full-time. As we get to the point of storing 10's of terabytes or more, it'll be something to consider but not when we have yet to breach 1 terabyte combined.

I then found out that Google was offering a $300 trial credit for their cloud offerings[^6], and I figured that free backups for the next year would work splendidly until we decided on where to pay for our backups to be stored. Duplicati and restic both support Google Cloud Storage out of the box, so all that was left to do was sign up for the developer account with my regular Google account, create some storage buckets (or grant the appropriate permissions to do so within the backup program), and then follow the instructions for each program[^7].

## Miscellaneous

I did have a brief period where I experimented with using minio[^8] on a Google Cloud Engine VPS, and then having Minio use Google Cloud Storage as it's backend. The thought behind this was that when I switched backends, I could do so without having to reconfigure each of the clients (which are spread across a number of US states). However, I ran into a handful of performance issues running minio on a cheap VPS and trying to use it with a slow Internet service. It might be something worth exploring at a later time, but it wasn't worth the headache for me at the moment.

The last caveat I ran into pertained to the aspect of restic that I previously mentioned, namely that restic relies on access to the repository to process, check, or prune backups. Restic needed to read through the entire repo to perform a few key operations, and with our Internet service the backup process would have taken more than 12 hours. There seems to be a local cache of meta data in the works that would mitigate this issue, but until that time it precluded us from using restic to backup directly to the cloud. The work around for this was using restic to create and maintain the second backup, and then using a tool called rclone[^17] to sync the local repository with a copy on the third party cloud provided.

Lastly, this setup isn't specific to my choice of tools at all. You can swap out restic or Duplicati with any number of backup clients, use a number of different tools instead of rclone to sync the remote repo with the local copy, and then utilize any number of third party hosts for your data. The important thing is that you sit down and actually do a backup.

## What to Backup

This probably won't be a hard process for you when considering what files to backup, since you will likely have a good idea of what you want saved from your computer. 

One thing I would recommend considering is what online services you use that you cannot afford to lose. A metaphorical ton of personal information is often stored and used on platforms like Facebook, GMail, Dropbox, etc. If there is anything that you couldn't stand to lose, then utilize services to back up those things. For me, I have several years worth of email stored in a personal GMail account, and Google is not above reproach when it comes to handling user accounts[^9]. Luckily, there is an excellent tool called Gmvault[^10] that allows for automated backups of any GMail account, on top of plenty of other features. Since I value the information in my GMail account, I back it up should Google ever decide to deactivate my account. It would still be incredibly annoying to have to fix that issue on the fly, but having the information backed up would keep it from being a catastrophic incident.

Other services, like the other Google services, Facebook, etc, may not have such a handy tool and may require periodic, manual backups stored locally which can then be backed up by your backup software of choice. Examples of this include Google's Takeout[^13] and Facebook's Download Your Information Tool[^14].

## The Process

As illustrated in the figure above, the process for backing up then is as follows:

1. Initialize a restic repository on a separate hard drive from your primary hard drives
2. (Optional) Since our Internet service is slow, I copied the local backup to a portable hard drive, took that to a place that has much better bandwidth, and seeded the remote backup from there.
3. Using a `systemd`[^11] timer[^12], perform the following steps nightly:
    1. Backup online services as needed
    2. Backup local files to the local, second backup
    3. Sync local backup to the cloud storage service
4. Using a `systemd`[^11] timer[^12], perform the following steps on a weekly (or maybe monthly) basis:
    1. Perform a full backup of online services (for example, a full Gmvault run instead of a quick run)
        * De-duplicate as necessary
    2. Backup local files to the local, second backup
    3. Sync local backup to the cloud storage service using rclone or something similar
    4. Perform health check of backups to ensure data is recoverable
5. Every once in a while, perform a partial or full restore of your data. This should also be done when you first begin using a backup system. This ensures that you understand the process of recovering data when you may be in a stressful state of mind, and tests the integrity of the backups. A backup that isn't tested isn't a backup at all.

## The Code

The following files are the templates for what I currently use. This was heavily inspired and built off of Erik Westrup's[^15] `restic-systemd-automatic-backup`[^16] GitHub project. There are two services, `nightly-backup` and `weekly-backup` that perform the steps outlined above. Each of those have two timers, which run the services nightly and weekly respectively, or in other words `nightly-backup` runs every night except for nights when `weekly-backup` runs. The services run Gmvault and rclone directly, while there are two scripts used when running restic. These files, `restic_backup.sh` and `restic_check.sh` run the backups, pruning, and checking as necessary. Finally, there are three auxiliary files `exclude.txt`, `restic_env.sh` , `restic_pw.txt` which each of the restic scripts source to determine what files to exclude in the backup, what parameters to use when running restic (namely, the location of the local repo and the password file), and the password used to encrypt the backup.

All of these can be found in my personal project here: [GitHub - seanlane - restic-systemd-automatic-backup](https://github.com/seanlane/restic-systemd-automatic-backup)

## Conclusion

I've been using these backups in their current form for a bit more than a week, and everything has been working great. The restore process from both backups has been flawless, every night a new snapshot is taken and then restic takes care of de-duplicating and pruning the snapshots according to my settings in `restic_env.sh`. I hope this post might be help to anyone looking to securely preserve their digital belongings on a budget.

## References

[^1]: [BackBlaze Blog: 3-2-1 Backup Strategy](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/)
[^2]: [Restic Home Page](https://restic.net/)
[^3]: [Duplicati Home Page](https://www.duplicati.com/)
[^4]: [Duplicati Forum: How long should a repair of a database take?](https://forum.duplicati.com/t/how-long-should-a-repair-of-a-database-take/1357/3)  
    [Repair fails, database recreate takes a ridiculously long time](https://forum.duplicati.com/t/repair-fails-database-recreate-takes-a-ridiculously-long-time/1799/7)  

[^5]: [GitHub - Restic: Issue #21](https://github.com/restic/restic/issues/21)
[^6]: [Google Cloud Free Trial](https://cloud.google.com/free/docs/frequently-asked-questions#free-trial)
[^7]: [Restic: Backup to GCS](http://restic.readthedocs.io/en/latest/030_preparing_a_new_repo.html#google-cloud-storage)  
    [Duplicati: Google Cloud Storage](https://duplicati.readthedocs.io/en/latest/05-storage-providers/#google-cloud-storage)  

[^8]: [Minio: Private Cloud Storage](https://minio.io/)
[^9]: [One Statistics Professor Was Just Banned By Google: Here Is His Story](http://investingchannel.com/article/433394/One-Statistics-Professor-Was-Just-Banned-By-Google-Here-Is-His-Story)  
    [How I lost access to my Google account today](https://ehsanakhgari.org/blog/2012-04-13/how-i-lost-access-my-google-account-today)  
    [Update No. 4: My @YouTube account was suspended for uploaded/embedded portions of AQ vids used for analysis for U.S. v Pfc. Manning. It appears my Google account is also slated for deletion.](https://alexaobrien.com/archives/3647)  
    [That time I got locked out of my Google account for a month](https://techcrunch.com/2017/12/22/that-time-i-got-locked-out-of-my-google-account-for-a-month/)  

[^10]: [Gmvault](http://gmvault.org/)
[^11]: [Arch Linux Wiki: Systemd](https://wiki.archlinux.org/index.php/systemd)
[^12]: [Arch Linux Wiki: systemd/Timers](https://wiki.archlinux.org/index.php/Systemd/Timers)
[^13]: [Google Takeout](https://takeout.google.com/)
[^14]: [Facebook: Accessing & Downloading Your Information](https://www.facebook.com/help/1701730696756992?helpref=hc_global_nav)
[^15]: [GitHub: Erik Westrup](https://github.com/erikw)
[^16]: [GitHub: restic-systemd-automatic-backup](https://github.com/erikw/restic-systemd-automatic-backup)
[^17]: [Rclone: Rsync for storage](https://rclone.org/)