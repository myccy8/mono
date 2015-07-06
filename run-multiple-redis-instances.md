* create a new redis .conf file

```Bash
$ cp /etc/redis.conf /etc/redis-xxx.conf
```

* edit /etc/redis-xxx.conf, illustrated as below

```Bash
...
#modify pidfile
#pidfile /var/run/redis/redis.pid
pidfile /var/run/redis/redis-xxx.pid

...
#dir /var/lib/redis/
dir /var/lib/redis-xxx/

...
#modify port
#port 6379
port 6380

...
#modify logfile
#logfile /var/log/redis/redis.log
logfile /var/log/redis/redis-xxx.log

...
#modify vm-swap-file
#vm-swap-file /tmp/redis.swap
vm-swap-file /tmp/redis-xxx.swap
...
```
* make dir /var/lib/redis-xxx

```Bash
$ mkdir -p /var/lib/redis-xxx
```

* copy init script

```Bash
$ cp /etc/init.d/redis /etc/init.d/redis-xxx
```

* edit the new init script

```Bash
...

#pidfile="/var/run/redis/redis.pid"
pidfile="/var/run/redis/redis-xxx.pid"

...

#REDIS_CONFIG="/etc/redis.conf"
REDIS_CONFIG="/etc/redis-xxx.conf"

...
```

* query the status of this redis in

```Bash
$ sudo service redis-xxx status
# server is stopped

# start service
$ sudo service redis-xxx start
```

* make redis-xxx service auto start

```Bash
$ sudo chkconfig --level 3 redis-xxx on
```