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

##### cp2 ReQL
######1
```
r.db('test').table('users').insert({"name":"k",age:20});
r.db('test').table('users').filter({name:"y"});
r.db('test').table('users').get("ee37e3b4-af45-4292-a6ef-8ed7167f3670").update({"age":2});
r.db('test').table('users').get("ee37e3b4-af45-4292-a6ef-8ed7167f3670").update({"newattr":100});
r.db('test').table('users').insert([{"name":"a",age:20},{"name":"b",age:30}]);
```
others:
```
r.db("dbname").table("name").filter().delete()
r.db("dbname").table("name").delete()
r.dbList()
r.dbDrop("dbname")
r.db("dbname").tableDrop("name")
```
##### cp3 clustering
######concept
vertical scalability: scaling up using stronger hardware
horizontal scalability: scaling out refers to the ability by adding hardware
######build culster
two ips: 10.0.0.1:29015  10.0.0.2:29015
edit 
```
vim /etc/rethinked/instance.d/default.conf
```

change:
```
bind=all
server-name=server1  (server2)
```
```
/etc/init.d/rethinkdb restart
```

######Replication:
command in book is incorrect.
```
r.db('test').table('users',{read_mode:'outdated'});
```


#####cp4 performance tuning
edit 
```
vim /etc/rethinked/instance.d/default.conf
```
edit
```
cache-size=2048
```

soft durability(less secure in case of power failure)
```
r.db('test').table('users').insert({"name":"k",age:20},{durability:'soft'});
```

######bulk data import
```
pip install rethinkdb
rethinkdb import -f data.json --table test.user
```

######create index
create a second index in tables->UI
```
r.db('test').table('users').getAll(10,{index:"age"})
```

create compound index
```
r.db('test').table('users').indexCreate("nameage",[r.row("name"),r.row("age")])
```
check status
```
r.db('test').table('users').indexStatus("nameage");
```
retrive data from compound index:
```
r.db('test').table('users').getAll(["k",20],{index:"nameage"})
```
