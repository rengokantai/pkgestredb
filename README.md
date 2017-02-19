# pkgsredb
## 1. Introducing RethinkDB
### Rethinking the database
#### Changefeeds
RethinkDB is designed for building real-time applications. Using a feature called Changefeeds, developers can program the database to continuously push data updates to applications in real time.  

#### Horizontal scalability
RethinkDB is a very good solution when flexibility and rapid iteration are of primary importance. Its other big strength is its ability to scale horizontally with very little effort.

### Installing RethinkDB
#### Installing RethinkDB on Ubuntu/Debian Linux
```
cat /etc/lsb-release
```
then
```
source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
apt update && apt install -y rethinkdb
```
#### install on centos(using AMI)
```
sudo wget http://download.rethinkdb.com/centos/7/`uname -m`/rethinkdb.repo -O /etc/yum.repos.d/rethinkdb.repo
sudo yum install rethinkdb
```
### Configuring RethinkDB
#### Running as a daemon
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

create a second index in tables->UI , or by command:
```
r.db('test').table('users').indexCreate("name")
```

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
######Advenced query
```
r.db("test").table("users").skip(1).limit(10)     //similar to mongo
```

orderBy
```
r.db('test').table('users').orderBy('age')
r.db('test').table('users').orderBy(r.desc('age'))
```

orderBy using index
```
r.db('test').table('users').orderBy({index:r.desc('age')})
```

return a sample element:
```
r.db('test').table('users').sample(1)
```

group and count  //count the occurance of name
```
r.db('test').table('users').group('name').count()
```

ungroup:  -> If forgot, will cause an error  e: Cannot convert NUMBER to SEQUENCE in:
```
r.db('test').table('users').group('name').count().ungroup().orderBy(r.desc('reduction')).limit(3)
```
avg,max.pluck
```
r.db('test').table('users').avg('age')
r.db('test').table('users').max('age').pluck('age')
```


##### cp5



###### cp6. administration
######tools:
backup entire cluster:
```
rethinkdb dump -c 10.0.0.1:28015 -f /var/back.tar
```

backup db and table
```
rethinkdb dump -e dbname.tbname -f /var/back.tar
```

restore:
```
rethinkdb restore file.rar
rethinkdb restore file.tar -c 10.0.0.1:28015 dbname:tbname
```

secure port: (hidden rethinkdb database)
```
r.db('rethinkdb').
r.db('rethinkdb').table('cluster_config').get('auth').update({auth_key:'pas'})
```
if delete password, set auth_key to null



##### 7 
######using rethink
get realtime data changefeeds
```
r.db("test").table("users").skip(1).limit(10).changes()
```
