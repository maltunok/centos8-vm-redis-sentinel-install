# install redis - all three instances

dnf install @redis

systemctl start redis
systemctl enable redis
systemctl status redis

ss -ltpn | grep redis-server

# configure master

sudo nano /etc/redis.conf

bind 0.0.0.0
protected-mode no
supervised systemd
requirepass Passw0rd!
masterauth Passw0rd!

systemctl restart redis

# configure replicas

sudo nano /etc/redis.conf

bind 0.0.0.0
protected-mode no
supervised systemd
requirepass Passw0rd!
masterauth Passw0rd!
replicaof <MasterIP> 6379

systemctl restart redis

# firewall settings - all three instances

firewall-cmd --zone=public --permanent --add-port=6379/tcp
firewall-cmd --reload

# redis-sentinel - all three instances

systemctl start redis-sentinel
systemctl enable redis-sentinel
systemctl status redis-sentinel

sudo nano /etc/redis-sentinel.conf

sentinel monitor mymaster <MasterIP> 6379 2
sentinel auth-pass mymaster Passw0rd!
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 60000

protected-mode no

systemctl restart redis-sentinel

# firewall settings - all three instances

firewall-cmd --zone=public --permanent --add-port=26379/tcp
firewall-cmd --reload

# Check the Redis Sentinel Setup Status

redis-cli -p 26379 info sentinel

redis-cli -p 26379 sentinel master mymaster

redis-cli -p 26379 sentinel get-master-addr-by-name mymaster

# test the failover

connect to master

systemctl stop redis
