#### pkgestredb
#####1

######install on centos(using AMI)
```
sudo wget http://download.rethinkdb.com/centos/7/`uname -m`/rethinkdb.repo \
          -O /etc/yum.repos.d/rethinkdb.repo
sudo yum install rethinkdb
```

check status:
```
sudo /etc/init.d/rethinkdb status
```
```
sudo cp /etc/rethinkdb/default.conf.sample /etc/rethinkdb/instances.d/instance1.conf
sudo nano /etc/rethinkdb/instances.d/instance1.conf
```


edit file,edit:
```
bind=all
join=192.168.1.100:29015
```

start:
```
sudo /etc/init.d/rethinkdb start
```

#####2
######1
```
r.db('test').table('users').filter({name:"y"});
r.db('test').table('users').get("ee37e3b4-af45-4292-a6ef-8ed7167f3670");
