---
date: "2024-10-05"
tags: ["mongodb", "migration", "database", "mongodump", "mongorestore"]
title: "Migrating MongoDB with mongo[dump/restore] "
---

## Context

A few years ago I built this web service to track the vaccination of my pets. I have four pets and it makes it much easier to have a system to track all of their vaccines and send me an e-mail when they are due. I shared [VacinaPet](https://vacinapet.com/) with family and friends but the truth is that only my mom and I were using the service. As long as it is useful to someone, I will have the VacinaPet up and running, consequently generating some extra monthly expenses. 

Time has passed and I came to the conclusion that I was paying too much for hosting and had to find a quick solution to lower the costs. Most of the hosting costs came from the XS MongoDB instance that I wasn't using to its fullest. I then decided to move the service to a shared Dev MongoDB instance, losing some of the benefits from the dedicated instance but having enough of what I needed **for free**.  

I already had a Dev MongoDB instance running with an old version of VacinaPet's database and some old data from the time I first published the service. All I had to do was to backup and restore the Dev instance with the “production” data. I used the commands `mongodump` to backup and `mongorestore` to restore data and, in this article, I am sharing the step-by-step of how I did it. 

For whoever is curious about which hosting I am using, VacinaPet is fully hosted on [CleverCloud](https://www.clever-cloud.com/). 

## Installation

<aside>

The installation here described is for MacOS users. For other installation options see [Installing the Database Tools](https://www.mongodb.com/docs/database-tools/installation/installation/).

</aside>

First of all, `mongodump` and `mongorestore` have to be installed through the installation of the [MongoDB Database Tools](https://www.mongodb.com/docs/database-tools/) suite. For that, I used [Homebrew](https://brew.sh/):

```bash
brew tap mongodb/brew 
brew install mongodb-database-tools
```

## mongodump

This was my first time using `mongodump`, so I ran `mongodump --help` to identify my options: 

```bash
mongodump --help
```

{{< expandable label="Expand to see the output" level="2" >}}
```
Usage:
mongodump <options> <connection-string>

Export the content of a running server into .bson files.

Specify a database with -d and a collection with -c to only dump that database or collection.

Connection strings must begin with mongodb:// or mongodb+srv://.

See http://docs.mongodb.com/database-tools/mongodump/ for more information.

general options:
      --help                                                print usage
      --version                                             print the tool version and exit
      --config=                                             path to a configuration file

verbosity options:
-v, --verbose=<level>                                     more detailed log output (include multiple times for more verbosity, e.g. -vvvvv, or specify a numeric value, e.g. --verbose=N)
      --quiet                                               hide all log output

connection options:
-h, --host=<hostname>                                     mongodb host to connect to (setname/host1,host2 for replica sets)
      --port=<port>                                         server port (can also use --host hostname:port)

ssl options:
      --ssl                                                 connect to a mongod or mongos that has ssl enabled
      --sslCAFile=<filename>                                the .pem file containing the root certificate chain from the certificate authority
      --sslPEMKeyFile=<filename>                            the .pem file containing the certificate and key
      --sslPEMKeyPassword=<password>                        the password to decrypt the sslPEMKeyFile, if necessary
      --sslCRLFile=<filename>                               the .pem file containing the certificate revocation list
      --sslFIPSMode                                         use FIPS mode of the installed openssl library
      --tlsInsecure                                         bypass the validation for server's certificate chain and host name

authentication options:
-u, --username=<username>                                 username for authentication
-p, --password=<password>                                 password for authentication
      --authenticationDatabase=<database-name>              database that holds the user's credentials
      --authenticationMechanism=<mechanism>                 authentication mechanism to use
      --awsSessionToken=<aws-session-token>                 session token to authenticate via AWS IAM

kerberos options:
      --gssapiServiceName=<service-name>                    service name to use when authenticating using GSSAPI/Kerberos (default: mongodb)
      --gssapiHostName=<host-name>                          hostname to use when authenticating using GSSAPI/Kerberos (default: <remote server's address>)

namespace options:
-d, --db=<database-name>                                  database to use
-c, --collection=<collection-name>                        collection to use

uri options:
      --uri=mongodb-uri                                     mongodb uri connection string

query options:
-q, --query=                                              query filter, as a v2 Extended JSON string, e.g., '{"x":{"$gt":1}}'
      --queryFile=                                          path to a file containing a query filter (v2 Extended JSON)
      --readPreference=<string>|<json>                      specify either a preference mode (e.g. 'nearest') or a preference json object (e.g. '{mode: "nearest", tagSets: [{a: "b"}], maxStalenessSeconds: 123}')
      --forceTableScan                                      force a table scan (do not use $snapshot or hint _id). Deprecated since this is default behavior on WiredTiger

output options:
-o, --out=<directory-path>                                output directory, or '-' for stdout (default: 'dump')
      --gzip                                                compress archive or collection output with Gzip
      --oplog                                               for taking a point-in-time snapshot on a replica set that is not part of a sharded cluster.
      --archive=<file-path>                                 dump as an archive to the specified path. If flag is specified without a value, archive is written to stdout
      --dumpDbUsersAndRoles                                 dump user and role definitions for the specified database
      --excludeCollection=<collection-name>                 collection to exclude from the dump (may be specified multiple times to exclude additional collections)
      --excludeCollectionsWithPrefix=<collection-prefix>    exclude all collections from the dump that have the given prefix (may be specified multiple times to exclude additional prefixes)
-j, --numParallelCollections=                             number of collections to dump in parallel
      --viewsAsCollections                                  dump views as normal collections with their produced data, omitting standard collections 
```
{{< /expandable >}}  


Since I have easy access to the MongoDB URI from both of the instances, I used the `--uri` flag to make things simple: 

```bash
mongodump --uri=<redacted>
```

The following is the output of running the `mongodump` command on the XL instance:

```bash
mongodump --uri=<redacted>
2024-10-05T12:57:59.582+0200	WARNING: On some systems, a password provided directly in a connection string or using --uri may be visible to system status programs such as `ps` that may be invoked by other users. Consider omitting the password to provide it via stdin, or using the --config option to specify a configuration file with the password.
2024-10-05T12:58:00.319+0200	writing <redacted>.system to dump/<redacted>/system.bson
2024-10-05T12:58:00.345+0200	writing <redacted>.users to dump/<redacted>/users.bson
2024-10-05T12:58:00.371+0200	writing <redacted>.pets to dump/<redacted>/pets.bson
2024-10-05T12:58:00.424+0200	writing <redacted>.jobs to dump/<redacted>/jobs.bson
2024-10-05T12:58:00.476+0200	done dumping <redacted>.system (0 documents)
2024-10-05T12:58:00.485+0200	done dumping <redacted>.users (53 documents)
2024-10-05T12:58:00.511+0200	done dumping <redacted>.pets (52 documents)
2024-10-05T12:58:00.528+0200	done dumping <redacted>.jobs (80 documents)
```

This command generated some `json` and `bson` under the directory `dump/<db_name>` and it took less than a second to complete for this very small database. 

I repeated the same process with the Dev instance, just because I wanted to backup the old data before migrating the production data to it. At the end of this step, I had the backup of the two MongoDB instances under `dump` folder.

## mongorestore

Next, it was time to restore the backed-up data from the XL instance to the Dev instance. Again, the first time using the command, I ran `mongorestore --help` to identify my options. 

```bash
mongorestore --help
```

{{< expandable label="Expand to see the output" level="2" >}}
```
Usage:
mongorestore <options> <connection-string> <directory or file to restore>

Restore backups generated with mongodump to a running server.

Specify a database with -d to restore a single database from the target directory,
or use -d and -c to restore a single collection from a single .bson file.

Connection strings must begin with mongodb:// or mongodb+srv://.

See http://docs.mongodb.com/database-tools/mongorestore/ for more information.

general options:
      --help                                                print usage
      --version                                             print the tool version and exit
      --config=                                             path to a configuration file

verbosity options:
-v, --verbose=<level>                                     more detailed log output (include multiple times for more verbosity, e.g. -vvvvv, or specify a numeric value, e.g. --verbose=N)
      --quiet                                               hide all log output

connection options:
-h, --host=<hostname>                                     mongodb host to connect to (setname/host1,host2 for replica sets)
      --port=<port>                                         server port (can also use --host hostname:port)

ssl options:
      --ssl                                                 connect to a mongod or mongos that has ssl enabled
      --sslCAFile=<filename>                                the .pem file containing the root certificate chain from the certificate authority
      --sslPEMKeyFile=<filename>                            the .pem file containing the certificate and key
      --sslPEMKeyPassword=<password>                        the password to decrypt the sslPEMKeyFile, if necessary
      --sslCRLFile=<filename>                               the .pem file containing the certificate revocation list
      --sslFIPSMode                                         use FIPS mode of the installed openssl library
      --tlsInsecure                                         bypass the validation for server's certificate chain and host name

authentication options:
-u, --username=<username>                                 username for authentication
-p, --password=<password>                                 password for authentication
      --authenticationDatabase=<database-name>              database that holds the user's credentials
      --authenticationMechanism=<mechanism>                 authentication mechanism to use
      --awsSessionToken=<aws-session-token>                 session token to authenticate via AWS IAM

kerberos options:
      --gssapiServiceName=<service-name>                    service name to use when authenticating using GSSAPI/Kerberos (default: mongodb)
      --gssapiHostName=<host-name>                          hostname to use when authenticating using GSSAPI/Kerberos (default: <remote server's address>)

namespace options:
-d, --db=<database-name>                                  database to use
-c, --collection=<collection-name>                        collection to use

uri options:
      --uri=mongodb-uri                                     mongodb uri connection string

namespace options:
      --excludeCollection=<collection-name>                 DEPRECATED; collection to skip over during restore (may be specified multiple times to exclude additional collections)
      --excludeCollectionsWithPrefix=<collection-prefix>    DEPRECATED; collections to skip over during restore that have the given prefix (may be specified multiple times to exclude additional prefixes)
      --nsExclude=<namespace-pattern>                       exclude matching namespaces
      --nsInclude=<namespace-pattern>                       include matching namespaces
      --nsFrom=<namespace-pattern>                          rename matching namespaces, must have matching nsTo
      --nsTo=<namespace-pattern>                            rename matched namespaces, must have matching nsFrom

input options:
      --objcheck                                            validate all objects before inserting
      --oplogReplay                                         for recovering a point-in-time snapshot on a replica set that is not part of a sharded cluster.
      --oplogLimit=<seconds>[:ordinal]                      only include oplog entries before the provided Timestamp
      --oplogFile=<filename>                                oplog file to use for replay of oplog
      --archive=<filename>                                  restore dump from the specified archive file.  If flag is specified without a value, archive is read from stdin
      --restoreDbUsersAndRoles                              restore user and role definitions for the given database
      --dir=<directory-name>                                input directory, use '-' for stdin
      --gzip                                                decompress gzipped input

restore options:
      --drop                                                drop each collection before import
      --dryRun                                              view summary without importing anything. recommended with verbosity
      --writeConcern=<write-concern>                        write concern options e.g. --writeConcern majority, --writeConcern '{w: 3, wtimeout: 500, fsync: true, j: true}'
      --noIndexRestore                                      don't restore indexes
      --convertLegacyIndexes                                Removes invalid index options and rewrites legacy option values (e.g. true becomes 1).
      --noOptionsRestore                                    don't restore collection options
      --keepIndexVersion                                    don't update index version
      --maintainInsertionOrder                              restore the documents in the order of their appearance in the input source. By default the insertions will be performed in an arbitrary order. Setting this flag also enables the
                                                            behavior of --stopOnError and restricts NumInsertionWorkersPerCollection to 1.
-j, --numParallelCollections=                             number of collections to restore in parallel
      --numInsertionWorkersPerCollection=                   number of insert operations to run concurrently per collection
      --stopOnError                                         halt after encountering any error during insertion. By default, mongorestore will attempt to continue through document validation and DuplicateKey errors, but with this option
                                                            enabled, the tool will stop instead. A small number of documents may be inserted after encountering an error even with this option enabled; use --maintainInsertionOrder to halt
                                                            immediately after an error
      --bypassDocumentValidation                            bypass document validation
      --preserveUUID                                        preserve original collection UUIDs (off by default, requires drop)
      --fixDottedHashIndex                                  when enabled, all the hashed indexes on dotted fields will be created as single field ascending indexes on the destination
```
{{< /expandable >}}    

Interestingly enough, they have a `--dryRun` option to view a summary of the restore process without actually restoring anything. Since this was my first time doing this, dry running was a great option to understand how this thing works before running the command for real. 

Like with `mongodump`, I used `--uri` to make things simple. I also decided to include `--maintainInsertionOrder`, because the existing order would help me validate the restore, and  `--drop` to clean up the old data from the collections. The final command was:

```bash
mongorestore --uri=<redacted> --dryRun --drop --maintainInsertionOrder --dir=dump/<xl_db_name>
```

<aside>
💡

This is a very small database so I am not worried at all about performance. If you have performance concerns, I recommend understanding better the other options under `restore options:`.

</aside>

The output of the dry run was very simple and didn't give me too much information: 
```bash
➜ mongorestore --uri=<redacted> --dryRun --drop --maintainInsertionOrder --dir=dump/<xl_db_name>
2024-10-05T13:41:25.982+0200	The --db and --collection flags are deprecated for this use-case; please use --nsInclude instead, i.e. with --nsInclude=${DATABASE}.${COLLECTION}
2024-10-05T13:41:25.983+0200	building a list of collections to restore from dump/<xl_db_name> dir
2024-10-05T13:41:25.984+0200	dry run completed
2024-10-05T13:41:25.984+0200	0 document(s) restored successfully. 0 document(s) failed to restore.
```

I rerun with the `--verbose` option, which gave me a little more visibility about the process: 
```bash
➜ mongorestore --uri=<redacted> --dryRun --drop --maintainInsertionOrder --dir=dump/<xl_db_name> --verbose
2024-10-05T13:42:25.478+0200	using --dir flag instead of arguments
2024-10-05T13:42:25.478+0200	using write concern: &{majority false 0}
2024-10-05T13:42:26.408+0200	The --db and --collection flags are deprecated for this use-case; please use --nsInclude instead, i.e. with --nsInclude=${DATABASE}.${COLLECTION}
2024-10-05T13:42:26.408+0200	building a list of collections to restore from <redacted> dir
2024-10-05T13:42:26.409+0200	found collection <redacted>.jobs bson to restore to <redacted>.jobs
2024-10-05T13:42:26.409+0200	found collection metadata from <redacted>.jobs to restore to <redacted>.jobs
2024-10-05T13:42:26.409+0200	found collection <redacted>.pets bson to restore to <redacted>.pets
2024-10-05T13:42:26.409+0200	found collection metadata from <redacted>.pets to restore to <redacted>.pets
2024-10-05T13:42:26.409+0200	found collection <redacted>.system bson to restore to <redacted>.system
2024-10-05T13:42:26.409+0200	found collection metadata from <redacted>.system to restore to <redacted>.system
2024-10-05T13:42:26.409+0200	found collection <redacted>.users bson to restore to <redacted>.users
2024-10-05T13:42:26.409+0200	found collection metadata from <redacted>.users to restore to <redacted>.users
2024-10-05T13:42:26.409+0200	dry run completed
2024-10-05T13:42:26.409+0200	0 document(s) restored successfully. 0 document(s) failed to restore.
```

After that, I ran the command for real, without the `--dryRun` option, and I surprisingly got an error, which I expanded on in the [Troubleshooting](#troubleshooting) section.  

Rerunning the command with the correct information resulted in what it seemed to be a successful date restore! 
```bash
mongorestore --uri=<redacted> --drop --maintainInsertionOrder --dir=dump/<redacted>
2024-10-05T13:45:01.088+0200	The --db and --collection flags are deprecated for this use-case; please use --nsInclude instead, i.e. with --nsInclude=${DATABASE}.${COLLECTION}
2024-10-05T13:45:01.089+0200	building a list of collections to restore from <redacted> dir
2024-10-05T13:45:01.090+0200	reading metadata for <redacted>.jobs from <redacted>/jobs.metadata.json
2024-10-05T13:45:01.090+0200	reading metadata for <redacted>.pets from <redacted>/pets.metadata.json
2024-10-05T13:45:01.091+0200	reading metadata for <redacted>.system from <redacted>/system.metadata.json
2024-10-05T13:45:01.091+0200	reading metadata for <redacted>.users from <redacted>/users.metadata.json
2024-10-05T13:45:01.145+0200	dropping collection <redacted>.pets before restoring
2024-10-05T13:45:01.145+0200	dropping collection <redacted>.system before restoring
2024-10-05T13:45:01.145+0200	dropping collection <redacted>.users before restoring
2024-10-05T13:45:01.209+0200	restoring <redacted>.jobs from <redacted>/jobs.bson
2024-10-05T13:45:01.325+0200	restoring <redacted>.pets from <redacted>/pets.bson
2024-10-05T13:45:01.335+0200	restoring <redacted>.users from <redacted>/users.bson
2024-10-05T13:45:01.355+0200	restoring <redacted>.system from <redacted>/system.bson
2024-10-05T13:45:01.367+0200	finished restoring <redacted>.system (0 documents, 0 failures)
2024-10-05T13:45:01.385+0200	finished restoring <redacted>.pets (52 documents, 0 failures)
2024-10-05T13:45:01.403+0200	finished restoring <redacted>.users (53 documents, 0 failures)
2024-10-05T13:45:01.403+0200	finished restoring <redacted>.jobs (80 documents, 0 failures)
2024-10-05T13:45:01.404+0200	no indexes to restore for collection <redacted>.jobs
2024-10-05T13:45:01.404+0200	no indexes to restore for collection <redacted>.pets
2024-10-05T13:45:01.404+0200	no indexes to restore for collection <redacted>.system
2024-10-05T13:45:01.404+0200	no indexes to restore for collection <redacted>.users
2024-10-05T13:45:01.404+0200	185 document(s) restored successfully. 0 document(s) failed to restore.
```

Remember that I wanted to keep the order of data? After restoring, I compared the data in both instances and, because they were ordered the same, I was able to quickly validate that they looked the same. Besides that, I also visited VacinaPet and validated that my user and all of its updated data were in there. 🎉

## Conclusion

It was worth taking a Saturday morning to do a quick learning of how to backup and restore MongoDB databases, do it on my service, and start paying 5x less while still having this emotional app up and running, and remembering about my pets’ vaccination. ❣️

If you read until here, I appreciate your time! Thank you 🤗

## Appendix

### Troubleshooting

When running `mongorestore` for the first time, I got `error running create command: (NotMaster) not master`. 

```bash
➜ mongorestore --uri=<redacted> --drop --maintainInsertionOrder --dir=dump/<redacted>
2024-10-05T13:44:31.191+0200	The --db and --collection flags are deprecated for this use-case; please use --nsInclude instead, i.e. with --nsInclude=${DATABASE}.${COLLECTION}
2024-10-05T13:44:31.192+0200	building a list of collections to restore from dump/<redacted> dir
2024-10-05T13:44:31.193+0200	reading metadata for <redacted>.jobs from dump/<redacted>/jobs.metadata.json
2024-10-05T13:44:31.193+0200	reading metadata for <redacted>.pets from dump/<redacted>/pets.metadata.json
2024-10-05T13:44:31.194+0200	reading metadata for <redacted>.system from dump/<redacted>/system.metadata.json
2024-10-05T13:44:31.194+0200	reading metadata for <redacted>.users from dump/<redacted>/users.metadata.json
2024-10-05T13:44:31.220+0200	dropping collection <redacted>.pets before restoring
2024-10-05T13:44:31.220+0200	dropping collection <redacted>.users before restoring
2024-10-05T13:44:31.220+0200	dropping collection <redacted>.system before restoring
2024-10-05T13:44:31.246+0200	finished restoring <redacted>.jobs (0 documents, 0 failures)
2024-10-05T13:44:31.246+0200	Failed: <redacted>.jobs: error creating collection <redacted>.jobs: error running create command: (NotMaster) not master
2024-10-05T13:44:31.246+0200	0 document(s) restored successfully. 0 document(s) failed to restore.
2024-10-05T13:44:31.246+0200	finished restoring <redacted>.users (0 documents, 0 failures)
```

I quickly Googled it and clicked on the [first stackoverflow link](https://stackoverflow.com/questions/60063070/mongorestore-keeps-throwing-notmaster-not-master-error-at-me) where someone was saying “*"NotMaster" usually means you are connected to a secondary node that cannot take writes”.* OOPS, I guess I used the wrong URI! 

The Dev MongoDB uses a replica set and has two nodes, one works as a primary node, accepting write operations, and the other is the secondary node that copies the data from the primary one and serves read queries. I used the secondary node URI to dump (read) the data and tried to use the same URI to restore (write), which didn't work. To fix this error, I just had to update the command `--uri` to the primary node.

On this topic, both `mongodump` and `mongorestore` don't accept URI containing all the hosts from a replica set, for example, `mongodb://host1:27017,host2:27017/?replicaSet=rs0`.

### Extra

Since I am not always using MongoDB and am more used to Postgres, this [MongoDB Cheat Sheet](https://gist.github.com/bradtraversy/f407d642bdc3b31681bc7e56d95485b6) is always very helpful in refreshing my memory and getting me back on speed when working on VacinaPet.