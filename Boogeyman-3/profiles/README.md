#Profile short 

###What is the exposed root password? Ftrccw45PHyq
i used below command even the second part is not neccesary 
`vol -f linux.mem linux.bash.Bash | tee bash.txt`

###And what time was the users.db file approximately accessed? Format is YYYY-MM-DD HH:MM:SS? 2023-11-07 03:49:45
the asnwer is the same line of above question's answer

###What is the MD5 hash of the malicious file found? 0511ccaad402d6d13ce801e1e9136ba2
You already have a clue from linux.bash.Bash: wget ... shell.c, compiled to pkexecc, then executed. So the likely malicious file is pkexecc; now you need to locate/dump it and hash it.

the run this code:
```
`mkdir -p dumps

vol -f linux.mem -o dumps linux.pagecache.InodePages --find /home/paco/pkexecc --dump | tee inode_pkexecc.txt
md5sum dumps/inode_*.dmp
```

###What is the IP address and port of the malicious actor? Format is IP:Port? 10.0.2.72:1337
from the previous output which was related to download the shell we know the malicious ip and now we have to find the port with below command:
`vol -f linux.mem linux.sockstat.Sockstat | grep "10.0.2.72"`

###What is the full path of the cronjob file and its inode number? Format is filename:inode number? /var/spool/cron/crontabs/root:131127
i used below commands:
`vol -f linux.mem linux.pagecache.Files | grep -iE "cron|crontab|crontabs"`
![inode](inode.png)


###What command is found inside the cronjob file? * * * * * cp /opt/.bashrc /root/.bashrc
with this command:
`vol -f linux.mem -o dumps linux.pagecache.InodePages --find /var/spool/cron/crontabs/root --dump` and then `cat dumps/*` Or
`strings -a linux.mem | grep -A3 -B3 "cp /opt/.bashrc"` for faster	
