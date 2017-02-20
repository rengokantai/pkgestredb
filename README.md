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
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -
apt update && apt install -y rethinkdb
```
#### install on centos(using AMI)
```
sudo wget http://download.rethinkdb.com/centos/7/`uname -m`/rethinkdb.repo -O /etc/yum.repos.d/rethinkdb.repo
sudo yum install rethinkdb
```
### Configuring RethinkDB
#### Running as a daemon
check status:(this seems not working)
```
sudo /etc/init.d/rethinkdb status
```
instead use this:
```
service rethinkdb status
```
```
cp /etc/rethinkdb/default.conf.sample /etc/rethinkdb/instances.d/instance1.conf
vim /etc/rethinkdb/instances.d/instance1.conf
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
u16
```
service rethinkdb restart
```
### Running a query
```
r.db('test').tableCreate('people')
```

## 2. The ReQL Query Language
### Documents
#### JSON document format
##### Embedded documents
```
r.db("test").table("users")("address")("postcode")
```
### Introducing ReQL
orderBy
```
r.table('users').filter({ name: 'Alex' }).orderBy(r.desc('age'))
```
### Inserting data

#### Batch inserts
```
r.db('test').table('people').insert([{document1}, {document2}])
```
```
r.db('test').table('users').insert({"name":"k",age:20});
r.db('test').table('users').filter({name:"y"});
r.db('test').table('users').get("ee37e3b4-af45-4292-a6ef-8ed7167f3670").update({"age":2});
r.db('test').table('users').get("ee37e3b4-af45-4292-a6ef-8ed7167f3670").update({"newattr":100});
r.db('test').table('users').insert([{"name":"a",age:20},{"name":"b",age:30}]);
```
### Reading data
#### Filtering results
```
r.table('people').filter(r.row("address")("city").eq("London"))
```
#### Manipulating results
```
r.table('people').filter(r.row("yearOfBirth").ge(1990)).pluck("email","phone")
```



### Deleting data
#### Removing all documents
```
r.db("dbname").table("name").filter().delete()
r.db("dbname").table("name").delete()
r.dbList()
```
#### Deleting a database
```
r.dbDrop("dbname")
r.db("dbname").tableDrop("name")
```
##3. Clustering, Sharding, and Replication
### An introduction to scaling
vertical scalability: scaling up using stronger hardware
horizontal scalability: scaling out refers to the ability by adding hardware
### Clustering RethinkDB
#### Adding a server to the cluster
two ips: 10.0.0.1:29015  10.0.0.2:29015
edit 
```
vim /etc/rethinked/instance.d/default.conf
```
If you are lazy,use

```
vim /etc/rethinked/instance.d/insrance1.conf
```
we created before.
change:
```
bind=all
server-name=server1  (server2)
```
```
/etc/init.d/rethinkdb restart
```

###Replication:
Read table's data if a table is offline:
```
r.db('test').table('users',{read_mode:'outdated'});
```


##4. Performance Tuning and Advanced Queries
### Performance tuning
#### Increasing the cache size
edit 
```
vim /etc/rethinked/instance.d/default.conf
```
edit
```
cache-size=2048
```
#### Increasing concurrency
It is slow it insert one by one
```
r.db('test').table('test').insert({"name": "Alex"})
r.db('test').table('test').insert({"name": "Louise"})
r.db('test').table('test').insert({"name": "Matt"})
```
insert many at once:(faster)
```
r.db('test').table('test').insert([{"name": "Alex"}, {"name": "Louise"}, {"name": "Matt"}])
```
####Using soft durability mode
soft durability(less secure in case of power failure)
```
r.db('test').table('users').insert({"name":"k",age:20},{durability:'soft'});
```

###bulk data import
```
apt install python-pip
pip install rethinkdb
rethinkdb import -f data.json --table test.user
```
### Introducing indexing
A query that doesn't make use of an index is called a full table scan, which means that the database has to go through the entire table to find the query result

### Evaluating query performance
Data Explorer -> Enable query profiler 

###Creating and using an index

create a second index in tables->UI , or by command:
```
r.db('test').table('users').indexCreate("name")
```

```
r.db('test').table('users').getAll(10,{index:"age"})
```
#### Compound indexes
filter
```
r.db('test').table('people').filter({name: "Alex", age: 22})
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
### Advenced queries
#### Limits, skips, and sorts
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
#### Finding a random document
return a sample element:
```
r.db('test').table('users').sample(1)
```
#### Grouping
group and count  //count the occurance of name
```
r.db('test').table('users').group('name').count()
```

ungroup:  -> If forgot, will cause an error  e: Cannot convert NUMBER to SEQUENCE in:
```
r.db('test').table('users').group('name').count().ungroup().orderBy(r.desc('reduction')).limit(3)
```

#### Aggregations
avg,max.pluck
```
r.db('test').table('users').avg('age')
r.db('test').table('users').max('age').pluck('age')
```


##5. Programming RethinkDB in Node.js



##6. RethinkDB Administration and Deployment
### RethinkDB administration tools
######default command
```
rethinkdb dump
```
By default, running the command without any arguments will connect to the database node on localhost—port 28015—and this will save the backup as a TAR archive named rethinkdb_dump followed by the date and time.
######backup entire cluster:
```
rethinkdb dump -c 10.0.0.1:28015 -f /var/back.tar
```
#### Backing up a single table
backup db and table
```
rethinkdb dump -e dbname.tbname -f /var/back.tar
```

#### Setting up automatic backups
(rethinkdb-dump = rethinkdb dump)  
```
/usr/bin/rethinkdb dump -f /home/Ubuntu/backup.tar
```

### Restoring your data
```
rethinkdb restore file.rar
rethinkdb restore file.tar -c 10.0.0.1:28015 -i dbname:tbname
```
### Securing RethinkDB
#### secure port: (hidden rethinkdb database)
block 8080 except localhost
```
iptables -A INPUT -i eth0 -p tcp --dport 8080 -j DROP
iptables -I INPUT -i eth0 -s 127.0.0.1 -p tcp --dport 8080 -j ACCEPT
```

#### Securing the driver port
```
r.db('rethinkdb').table('users').get('admin').update({password: 'pass'})
```
DO NOT USE THIS. Deprecated.
```
r.db('rethinkdb').table('cluster_config').get('auth').update({auth_key:'pass'})
```
if delete password, set password to null
### Monitoring RethinkDB
```
r.db("rethinkdb").table("current_issues").run(conn, callback);
```

```
r.db("rethinkdb").table("jobs").run(conn, callback);
```


##### 7 
######using rethink
get realtime data changefeeds
```
r.db("test").table("users").skip(1).limit(10).changes()
```
