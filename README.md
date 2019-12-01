# helpshiftdoc

1. Login into system using given key, faced key bad permission error and fixed using below command
   `chmod 600 key_name`

2. After Login, tried installing `redis-server` using `sudo apt-get install redis-server` and faced disk space issue.

3. I have ran `df -h` and found that `/` is 100% full.

4. Then I have try to get all top large files and checked size of `/` using below commands


```
sudo du -ah /home | sort -n -r | head -n 5
sudo find / -type f -exec du -Sh {} + | sort -rh | head -n 5
sudo du -cha --max-depth=1 / | grep -E "M|G"
```

```
   testuser@ip-172-33-4-192:/etc/init.d$ sudo du -cha --max-depth=1 / | grep -E "M|G"
    5.7M	/etc
    224M	/var
    9.6M	/mnt
    12M	/sbin
    62M	/lib
    25M	/boot
    550M	/usr
    9.6M	/bin
    897M	/
    897M	total    
```
    
   and findout that total used size of / is only 897M

   After looking at this output I thought that the issue is something else and then i googled and find out that space is not freed from disk even after deleting files

   `sudo lsof | grep deleted`
   
   Then i found this link and followed it

    https://access.redhat.com/solutions/2316

5. After this disk is freed I was able to install redis-server.

6. Now I tried to run redis-server but port was alredy and find out that a `nc` process is using the port

    `sudo netstat -lntp`    
    `sudo ps -ef | grep nc`

    then i have stopped the `nc` server process

6. Then I have started redis-server but not able to connect to redis-cli and after so many tries i have ran the server on different port 16379 and i was able to connect
    then i checked iptable rules and found the issue.
    
    I have flushed the rules from iptables since it was only one DROP rule.
    `sudo iptables -F`
    
    For permanent chnages
    
    `cat /etc/init.d/iptables-persistent`
    
    `cat /etc/iptables/rules.v4  -> edited the file and ACCPETED the 6379 port`

7. After running the `redis-server` I have created redis.conf at `/etc/redis/redis.conf`

```
# Redis configuration file example

daemonize yes

# When running daemonized, Redis writes a pid file in /var/run/redis.pid by
# default. You can specify a custom pid file location here.
pidfile /var/run/redis/redis-server.pid

# Accept connections on the specified port, default is 6379.
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379

bind 127.0.0.1

timeout 0

tcp-keepalive 0

loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile /mnt/redis/redis-server.log

databases 2
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes

rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb

# Note that you must specify a directory here, not a file name.
dir /mnt/redis

#requirepass password
```

8. and also created `init.d/redis_6379` script to start redis server with below content

```
#!/bin/sh

REDISPORT=6379
EXEC=/usr/bin/redis-server
CLIEXEC=/usr/bin/redis-cli

PIDFILE=/var/run/redis_server.pid
CONF="/etc/redis/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

9. After this i have run the redis server and set the keys


```
testuser@ip-172-33-4-192:~$ sudo /etc/init.d/redis_6379 start
Starting Redis server...
testuser@ip-172-33-4-192:~$ redis-cli set testkey 1
OK
testuser@ip-172-33-4-192:~$ redis-cli get testkey
"1"

```
