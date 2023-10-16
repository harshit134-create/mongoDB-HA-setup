# mongoDB-HA-setup

MONGODB
=======

-> MongoDB is a document based database designed for ease of application development and scaling.It stores data in a type of JSON format called BSON.
-> MongoDB a Nosql database.
-> So whenever you use a sql database/relational database , then you have a follow a certain schema i.e. Whichever the column is you have to fill the value
   in their rows , for example you have 2 columns named "name" and "age" then you have to fill the values in those 2 columns an dcan't go beyond making a
   new column for new entry say ID , but in case of mongoDB you can create one such record without worrying about the schema.
   Let me show you by example:
   MYSQL                                                               
   -----
   {                                                                      
	Name: "Harshit",
	Sport: "football",
	date: Date()
   }

   {
	Name: "David",
	Sport: "Cricket",
	date: Date(),
        ID: "1234"
   }

Now this is invalid bcoz you are not following a certain schema over here. But this is completely fine in mongoDB. i.e. you can't add a new column ID in case on mYSQL.
So all in all you can add additional fields in your mongoDB database without worrying about a schema unlike MYSQL.

-> Q) What is the difference between Mongo and mongoD ??
   Ans) Mongo is  a command line shell that connects to a specific instance of mongoD. Whereas mongoD is mongo Daemon , its basically a host process for Database.
-> Let's understand termnologies of MongoDB in easy language by comparison with MS excel and MySQL:
   Excel            MYSQL           mongoDB
   Workbook         Database        Database
   Sheets           Tables          Collections
   Rows             Rows            documents(BSON)
   Columns          Columns         Fields

-> Whenever you insert a document in mongoDB , you get one ObjectID and whenever you want to access this document you can use this objectID. ObectID is like primary key in MYSQL>

-> Records in a MongoDB database are called documents, and the field values may include numbers, strings, booleans, arrays, or even nested documents.

-> {
	Name: "Harshit",
	Sport: "football",
	date: Date()
   }

-> Follow the link below for the cheat sheet for mongoDB commands:
   https://www.codewithharry.com/blogpost/mongodb-cheatsheet/
 
-> Q). What is mongosh ?
   Ans) The MongoDB Shell, mongosh, is used for interacting with MongoDB deployments locally, or on another remote host. Use the MongoDB Shell to test queries and interact with the data in your MongoDB database.


mongoDB Replicaset
==================

-> A replica set in MongoDB is a group of mongod processes that provide redundancy and high availability.
-> The members of a replica set are:
   Primary - The primary receives all write operations.
   Secondaries - Secondaries replicate operations from the primary to maintain an identical data set.
-> The minimum recommended configuration for a replica set is a three member replica set with three data-bearing members: one primary and two secondary members.
-> The primary is the only member in the replica set that receives write operations. MongoDB applies write operations on the primary and then records the operations on the primary's oplog. Secondary members replicate this log and apply the operations to their data sets.
-> All members of the replica set can accept read operations. However, by default, an application directs its read operations to the primary member.
-> The replica set can have at most one primary. If the current primary becomes unavailable, an election determines the new primary.
->  A secondary maintains a copy of the primary's data set. To replicate data, a secondary applies operations from the primary's oplog to its own data set in an asynchronous process. [1] A replica set can have one or more secondaries.
-> Although clients cannot write data to secondaries, clients can read data from secondary members.
-> A secondary can become a primary. If the current primary becomes unavailable, the replica set holds an election to choose which of the secondaries becomes the new primary.
-> Arbiter - An arbiter participates in elections for primary but an arbiter does not have a copy of the data set and cannot become a primary.
  

Deploy a Highly available mongoDB replicaset
============================================

-> Three member replica sets provide enough redundancy to survive most network partitions and other system failures.These sets also have sufficient capacity for many distributed read operations. Replica sets should always have an odd number of members. This ensures that elections will proceed smoothly. 
-> For production deployments, you should maintain as much separation between members as possible by hosting the mongod instances on separate machines.That's why we will be using statefulset.
-> ensure that MongoDB listens on the default port of 27017.
-> Use the --bind_ip option to ensure that MongoDB listens for connections from applications on configured addresses. you should use --bind_ip_all option.
-> Configuration - Create the directory where MongoDB stores data files before deploying MongoDB. Specify the mongod configuration in a configuration file stored in /etc/mongod.conf or a related location.
-> Command to deploy a replicaset - mongod --replSet "rs0" --bind_ip localhost,<hostname(s)|ip address(es)>
-> Once your deployment is up and running go inside a pod and do the below activity.
-> # Then go inside one running pod and after that fire mongosh and you will enter the shell.
# Then fire rs.initiate( { _id : "rs0", members: [ { _id: 0, host: "mongodb0.example.net:27017" }, { _id: 1, host: "mongodb1.example.net:27017" }, { _id: 2, host: "mongodb2.example.net:27017" } ] })
# Finally you can check the status of mongodb replicaset using either rs.conf() or rs rs.status()
# Now to check wheteher the replivcation bw primary and secondary is ghappening or not, temporarily
# delete the pod and again go inside the container and you will observe that primary has been changed.
# So replication/syncing is happening.


mongoDB high avalaibility setup using helm
==========================================

-> command- helm install my-release-2 oci://registry-1.docker.io/bitnamicharts/mongodb --set architecture=replicaset,auth.enabled=true,auth.rootUser=root,auth.rootPassword=root123,auth.usernames={harshit},auth.passwords={abcd1234},auth.databases={mdbdb},auth.replicaSetKey=key123,replicaSetName=rs0,replicaSetHostnames=true,replicaCount=3,podManagementPolicy=OrderedReady,containerPorts.mongodb=27017,service.ports.mongodb=27017,persistence.enabled=true,persistence.size=4Gi,persistence.mountPath=/bitnami/mongodb,arbiter.enabled=false,volumePermissions.enabled=true,arbiter.podSecurityContext.enabled=false,arbiter.containerSecurityContext.enabled=false,arbiter.containerSecurityContext.runAsNonRoot=false,arbiter.livenessProbe.enabled=false,arbiter.readinessProbe.enabled=false,hidden.containerSecurityContext.enabled=false,hidden.containerSecurityContext.runAsNonRoot=false,hidden.livenessProbe.enabled=false,hidden.readinessProbe.enabled=false,hidden.persistence.enabled=false,terminationGracePeriodSeconds=20
-> Now go under one pod and enter the command "mongosh -u username" Then this will ask for password , hence authentication is done.

