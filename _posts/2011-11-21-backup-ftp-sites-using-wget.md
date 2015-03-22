---
layout: post
title: Backup FTP sites using wget
tags: [Backup, Ubuntu]
description:
---

I read somewhere that it's very easy to backup a site using wget on linux. That made me realize that, at the moment, I do not have any backups of my sites. The only backups that (hopefully) exist are the ones my shared host creates. So I figured it would be safe to have my own backups. Since I run my own Ubuntu home-server, already for a few years now, I have a backup target too.

The only problem left is: the databases. Luckily dasBlog does not use a database to store it's posts, but a XML datastore. This makes it less painful to make a backup, especially on shared hosting. My MySQL databases can be accessed externally, so they can be backuped using a (separate) script too. Currently I already do this for my e-mail database.

So, I created the following script:

{% highlight bash %}
#!/bin/bash  

cd "$(dirname "$0")"  

backupfile=$(date +%Y%m%d)  
savetodir=./backup-$(date +%Y)/  
host=ftp.example.com  
user=backup  
pass=  

if [ -d $host ]; then  
    rm -r $host  
fi  

wget --recursive --level=inf --quiet --user=$user --password=$pass ftp://$host/  

if [ $? -ne 0 ]; then  
    echo "wget faild during the backup of $host" >&2  
    exit 1  
fi  

tar -cf $backupfile.tar $host && gzip -c $backupfile.tar > $backupfile.tar.gz && rm $backupfile.tar  

if [ $? -ne 0 ]; then  
    echo "faild to create a tar.gz during the backup of $host" >&2  
    exit 1  
fi  

if [ ! -d $savetodir ]; then  
    mkdir $savetodir  
fi  

mv $backupfile.tar.gz $savetodir  

exit 0  
{% endhighlight %}

{% highlight bash %}
Fist a few variables are set, like the ftp site, username and password. I created a special backup user that only has read privileges, since no writing is required when making backups.

By default wget creates a directory named after the host, so I start with removing that directory and it's contents but only if it exists (from a previous backup).

This is follwed by the wget command that does a fully-recursive download (note the: --level=inf) after finishing the exit code of wget is checked for errors. The backup is then compressed and stored in a directory per-year.

Finally I scheduled a cronjob, to automatically create a backup once a week:

0 3 * * 3 boaz sh /mnt/array1/backup/n3rd.nl-sohosted/backup.sh  
{% endhighlight %}
