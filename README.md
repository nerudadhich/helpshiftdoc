# helpshiftdoc

1. Login into system using given key, faced key bad permission error and fixed using below command
   `chmod 600 key_name`

2. After Login, tried installing `redis-server` using `sudo apt-get install redis-server` and faced disk space issue.

3. I have ran `df -h` and found that `/` is 100% full.

4. Then I have try get all top large files and checked size of `/`

    sudo du -ah /home | sort -n -r | head -n 5
    sudo find / -type f -exec du -Sh {} + | sort -rh | head -n 5
    sudo du -cha --max-depth=1 / | grep -E "M|G"


    testuser@ip-172-33-4-192:/etc/init.d$ sudo du -cha --max-depth=1 / | grep -E "M|G"
    5.7M	/etc
    224M	/var
    9.6M	/mnt
    du: cannot access ‘/proc/1961/task/1961/fd/4’: No such file or directory
    du: cannot access ‘/proc/1961/task/1961/fdinfo/4’: No such file or directory
    du: cannot access ‘/proc/1961/fd/4’: No such file or directory
    du: cannot access ‘/proc/1961/fdinfo/4’: No such file or directory
    12M	/sbin
    62M	/lib
    25M	/boot
    550M	/usr
    9.6M	/bin
    897M	/
    897M	total

    and findout that total used size of / is only 897M

    After looking at this output I thought that the issue is something else and then i googled and find out that space is not freed from disk even after deleting files

    `sudo lsof | grep deleted`
    Then followed this link

    https://access.redhat.com/solutions/2316

5. After this I was able to install redis-server

6. Now I tried to run redis-server but port was alredy and find out that a `nc` process is using the port
    `sudo netstat -lntp`
    
    `sudo ps -ef | grep nc`

    then i have stopped the `nc` server process

6. Then I have started redis-server but not able to connect to redis-cli and after so many tries i have ran the server on different port 16379 and i was able to connect
    then i checked iptable rules and found the issue

    `cat /etc/init.d/iptables-persistent`
    
    `cat /etc/iptables/rules.v4  -> edited the file and ACCPETED the 6379 port`

7. After running the `redis-server` I have created redis.conf file and setup redis AUTH

8. and also created `init.d/redis_6379` script to start redis server
