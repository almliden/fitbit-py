# Mongodb troubleshooting

Error `mongodb.service: Main process exited, code=killed, status=9/KILL` from `sudo service mongod status`.

It was killed, but why?

Look at the logs, config tells where these are stored: `/etc/mongodb.conf`

Could be that one collection is too large for the system.

How can we find out? 

Try these steps: [https://stackoverflow.com/questions/63727782/why-mongod-service-is-killed-by-my-system] which was taken from here. [https://sondnpt00343.medium.com/how-to-fix-mongod-service-32dbbe51a4ee]

1. Firstly, stop mongodb if you restarted:
    `sudo systemctl stop mongod`
2. Next, I detected the database corruption:
    `sudo mongod --repair --dbpath /var/lib/mongodb`
3. Run commands:
    `chown -R mongodb:mongodb /var/lib/mongodb`
    `chown mongodb:mongodb /tmp/mongodb-27017.sock`
4. Finally:
    `sudo systemctl start mongod`

Where the first step might say that the db is too large for the system. When running this: `sudo mongod --repair --dbpath /mnt/wd2500/mongodb/lib`

You could get the following output:

```log
Sat Jan 29 13:59:50.342
Sat Jan 29 13:59:50.342 warning: 32-bit servers don't have journaling enabled by default. Please use --journal if you want durability.
Sat Jan 29 13:59:50.342
Sat Jan 29 13:59:50.374 [initandlisten] MongoDB starting : pid=15085 port=27017 dbpath=/mnt/wd2500/mongodb/lib 32-bit host=pif
Sat Jan 29 13:59:50.374 [initandlisten]
Sat Jan 29 13:59:50.374 [initandlisten] ** NOTE: This is a 32 bit MongoDB binary.
Sat Jan 29 13:59:50.374 [initandlisten] **       32 bit builds are limited to less than 2GB of data (or less with --journal).                                                                                                 26Cudevd-control.s                                        udev.service
Sat Jan 29 13:59:50.374 [initandlisten] **       Note that journaling defaults to off for 32 bit and is currently off.
Sat Jan 29 13:59:50.374 [initandlisten] **       See http://dochub.mongodb.org/core/32bit
Sat Jan 29 13:59:50.374 [initandlisten]
Sat Jan 29 13:59:50.374 [initandlisten] db version v2.4.14
Sat Jan 29 13:59:50.374 [initandlisten] git version: nogitversion
Sat Jan 29 13:59:50.374 [initandlisten] build info: Linux bm-wb-03 3.19.0-trunk-armmp #1 SMP Debian 3.19.1-1~exp1+plugwash1 (2015-03-28) armv7l BOOST_LIB_VERSION=1_58
Sat Jan 29 13:59:50.374 [initandlisten] allocator: system
Sat Jan 29 13:59:50.374 [initandlisten] options: { dbpath: "/mnt/wd2500/mongodb/lib", repair: true }
**************
You specified --repair but there are dirty journal files. Please
restart without --repair to allow the journal files to be replayed.
If you wish to repair all databases, please shutdown cleanly and
run with --repair again.
**************
Sat Jan 29 13:59:50.379 [initandlisten] exception in initAndListen: 12596 old lock file, terminating
Sat Jan 29 13:59:50.379 dbexit:
Sat Jan 29 13:59:50.379 [initandlisten] shutdown: going to close listening sockets...
Sat Jan 29 13:59:50.379 [initandlisten] shutdown: going to flush diaglog...
Sat Jan 29 13:59:50.379 [initandlisten] shutdown: going to close sockets...
Sat Jan 29 13:59:50.379 [initandlisten] shutdown: waiting for fs preallocator...
Sat Jan 29 13:59:50.379 [initandlisten] shutdown: closing all files...
Sat Jan 29 13:59:50.379 [initandlisten] closeAllFiles() finished
Sat Jan 29 13:59:50.379 dbexit: really exiting now
```

Basically the db goes down when trying to insert data. [https://stackoverflow.com/questions/62953180/mongodb-service-goes-down-while-inserting-data]

To get it up and running again, copy the collection to another and we can continue inserting data. 

Had no success with `sudo mongodump -p -u super -d helt -v -o mongodb_dump` which said incorrect authentication. Must go through the console instead and find another solution.

Follow these steps:

1. Connect to the instance with the username and table: 
  `mongo -p -u admin admin`
2. Use the correct db
  `use helt`
3. Show collections:
  `show collections`
4. Show users:
  `show users`
5. Copy database:
  `db.copyDatabase('helt', 'helt-archive-2022-01-29')`
6. Make sure you are working on the correct db:
  `db` which will output the db we are working on, should be: `helt-archive-2022-01-29`
7. If not, use the correct db:
  `use helt-archive-2022-01-29`
8. Verify that the collection is there:
  `show collections`
9. To change back and drop the collection, do this:
  `use helt`
10. Now we should be able to drop the collection:
  `db.heart.drop()`
11. Now we might have to recreate the collection in order to insert new rows again:
  `db.createCollection('heart')`

More on commands in mongodb here: [https://docs.mongodb.com/v2.4/reference/method/db.collection.drop/] for the version installed.
