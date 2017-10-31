# Enable authentication for Mongo
[![N|Solid](http://www.travelanium.com/th/images/travelanium-logo.jpg)](http://www.travelanium.com/)

#### 1. Create fresh mongo container
```sh
 docker run --name mongodb \
-v /opt/seamless/mongodb/mongod.conf:/etc/mongod.conf \
-v /opt/seamless/mongodb/certs/master/xxx-key-cert.pem:/etc/ssl/xxx-key-cert.pem \
-v /opt/seamless/mongodb/certs/ca-cert.pem:/etc/ssl/ca-cert.pem \
-v /opt/seamless/mongodb/data:/data/db \
-v /opt/seamless/mongodb/configdb:/data/configdb \
-v /opt/seamless/mongodb/log:/var/log/mongodb \
--restart=always \
--network=isolated_nw \
-d mongo:3.4.7 bin/sh -c "mongod --config /etc/mongod.conf"
```

#### 2. Create admin user.
Access to mongo shell.
```sh
docker exec -it mongodb mongo
```
Create admin user.
```javascript
db.createUser({
    user: "admin", 
    pwd: "abc123",
    roles: [ { role: "root", db: "admin" } ]
})
```

#### 3. Restart mongod with --auth
```sh
docker update --restart=no mongodb;
docker exec -it mongodb mongod --shutdown;
docker rm -f mongodb;
docker run --name mongodb \
-v /opt/seamless/mongodb/mongod.conf:/etc/mongod.conf \
-v /opt/seamless/mongodb/certs/master/xxx-key-cert.pem:/etc/ssl/xxx-key-cert.pem \
-v /opt/seamless/mongodb/certs/ca-cert.pem:/etc/ssl/ca-cert.pem \
-v /opt/seamless/mongodb/data:/data/db \
-v /opt/seamless/mongodb/configdb:/data/configdb \
-v /opt/seamless/mongodb/log:/var/log/mongodb \
--restart=always \
--network=isolated_nw \
-d mongo:3.4.7 bin/sh -c "mongod --config /etc/mongod.conf --auth"
```
#### 4. verify setting
Connect to mongo shell without authentication parameter.
```sh
docker exec -it mongodb mongo
```
And use admin command
```javascript
show dbs
```
Your mongo shell should display error message like this.
```sh
> show dbs
2017-09-01T10:20:36.325+0000 E QUERY    [thread1] Error: listDatabases failed:{
        "ok" : 0,
        "errmsg" : "not authorized on admin to execute command { listDatabases: 1.0 }",
        "code" : 13,
        "codeName" : "Unauthorized"
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
shellHelper.show@src/mongo/shell/utils.js:769:19
shellHelper@src/mongo/shell/utils.js:659:15
@(shellhelp2):1:1
```

#### 5. Connect to mongo shell with authentication param
```sh
docker exec -it mongodb mongo --port 27017 -u "admin" -p "abc123" --authenticationDatabase "admin"
```
And use admin command again.
```javascript
show dbs
```
Your mongo shell should display a result without any error message.
```sh
> show dbs
admin  0.000GB
local  0.000GB
```

#### 6. Create additional user
```javascript
use test
db.createUser({
    user: "myTester",
    pwd: "xyz123",
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)
```

#### 7. Verify new user permission
```sh
docker exec -it mongodb mongo -u "myTester" -p "xyz123" --authenticationDatabase "test"
```
```javascript
db.foo.insert( { x: 1, y: 1 } );
db.foo.find();
```
