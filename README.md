# kolla-migrator

Tool that poll the operation to remote server and applies them to local/remote server.

To see list of commands available use:

  $ go run main.go --help

## Typically this command will take the following form:

  $ go run main.go migrate --src "mongodb://localhost:27018" --src-username mongomigrate --src-password some-password --src-ssl=false --dst "mongodb://localhost:27019" --dst-username mongomigrate --dst-password some-password --dst-ssl=false


This command copies database entries from the mongod instance running on the host mongodb0.example.net and duplicates operations to the host mongodb1.example.net. If you do not need to keep the --src host running during the migration, consider using mongodump and mongorestore or another backup operation, which may be better suited to your operation.


###  Running and testing with docker

  $ docker run --name mongo1 -p 27018:27017 -d mongo mongod --logpath ./tmp/1.log --port 27017 --replSet rs
  $ docker run --name mongo2 -p 27019:27017 -d mongo mongod --logpath ./tmp/1.log --port 27017 --replSet rs

Legecy mongo running into docker container:

  $ docker exec -it mongo1 mongo admin
  > rs.initiate({_id: 'rs', members: [ {_id: 1, host:'localhost:27017'}]})
  > db.createUser({ user: 'mongooplog', pwd: 'some-password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
  > db.createRole( 
  { 
      role: "anyAction", 
      privileges: [ { 
          resource: { anyResource: true }, 
          actions: [ "anyAction" ] } ], 
      roles: []
  })
  > db.grantRolesToUser("mongooplog",["anyAction"])


Legecy mongo running onto host:
-------------------------------

$mongo
> db.createUser({ user: 'mongooplog', pwd: 'some-password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
> db.createRole(
{
    role: "anyAction",
    privileges: [ {
        resource: { anyResource: true },
        actions: [ "anyAction" ] } ],
    roles: []
})



start mongo migrate:
-------------------

  $ go run main.go migrate --src "mongodb://localhost:27018" --src-username mongomigrate --src-password some-password --src-ssl=false --dst "mongodb://localhost:27019" --dst-username mongomigrate --dst-password some-password --dst-ssl=false


Try adding, deleting and modify some records in mongo1 server on host/container and check if they persist in mongo2 server. You can also use the status command to check the record count in each cluster.

  $ go run main.go status --src "mongodb://localhost:27018" --src-username mongomigrate --src-password some-password --src-ssl=false --dst "mongodb://localhost:27019" --dst-username mongomigrate --dst-password some-password --dst-ssl=false


