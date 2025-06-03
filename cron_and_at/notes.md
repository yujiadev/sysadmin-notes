# Cron and At

The cron system is configured to check the /var/spool/cron directory for jobs by user.

It also incopreates jobs defined in the /etc/anacrontab file, based on the 0anacron script in the /etc/cron.hourly directory.

It also checks the schedule jobs form the computer described in the /etc/crontab file and in the /etc/cron.d directory.

```bash 
cat /etc/crontab

find /var/log/tasks/ -type f -mtime +7 -delete

@hourly /usr/bin/echo "this si another test" >> /home/yujia/textfile.txt
```

