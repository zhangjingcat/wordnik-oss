# wordnik open-source tools

## Overview
These are tools used to maintain a MongoDB deployment.

### To build
Requires apache ant 1.7 or greater, java 6:

<pre>
cd src/java
ant -f install-ivy # only needed if you don't have apache ivy installed
ant dist
</pre>

### To run
<pre>
cd dist (or wherever you unzipped the distribution)
./bin/run.sh <tool-class> <options>
</pre>


### To get tool options
Run any tool with a -? parameter to see the options:

<pre>./bin/run.sh com.wordnik.system.mongodb.SnapshotUtil -?
usage: SnapshotUtil
 -c : CSV collection string (prefix with ! to exclude)
 -d : database name
 -h : hostname
 -t : threads
 -o : output directory
 [-s : max file size in MB]
 [-Z : compress files]
 [-J : output in JSON (default is BSON)]
 [-u : username]
 [-p : password]
</pre>


### Tools included
<pre>com.wordnik.system.mongodb.SnapshotUtil</pre>
This is pretty straight forward, it's meant for taking backups of your data.  The differences between it and mongodump are:
* It splits files based on a configurable size
* It will let you select what you want to backup with inclusion and exclusion operators
* It will automatically gzip the files as it rotates them
* It supports a JSON export
* It runs a configurable thread pool so you can backup multiple collections simultaneously

<pre>com.wordnik.system.mongodb.RestoreUtil</pre>

Operates against either mongodump files or files made with the SnapshotUtil with either uncompressed or compressed bson files. Also supports inclusion/exclusion of files

<pre>com.wordnik.system.mongodb.IncrementalBackupUtil</pre>

This queries a master server's oplog and maintains a set of files which can be replayed against a snapshot of the database.  The procedure we use is to snapshot the db (either at the filesystem or with the tool) and apply the incremental changes created by this tool.

The tool looks for a file called "last_timestamp.txt" in the CWD.  This file sets a starting point for querying the oplog--it should contain a single line in the format:

<pre>[time-in-seconds]|[counter]</pre>

The [time-in-seconds] is the seconds since epoch, which you can grab from the OS or from a tool like this:

http://www.esqsoft.com/javascript_examples/date-to-epoch.htm

The counter should typically be set to 0.  As the tool runs, every operation flushed to disk will cause this file to be updated.  If you want to stop a running process, create a file called "stop.txt" in the CWD of the application.  It will cause the app to stop within one second.

<pre>com.wordnik.system.mongodb.ReplayUtil</pre>

Takes a series of files created by the <pre>IncrementalBackupUtil</pre> and replays them.  The tool allows applying the operations against alternate databases and collections.  It also supports skipping records which fall outside a specified timepoint, if for instance you want to roll back to a particular point in time.
