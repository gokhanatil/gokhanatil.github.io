---
title: "BBED Block Browser Editor For Oracle 11g"
layout: post
---

BBED (Block Browser Editor) is a tool for Oracle internal use, and it helps you to read and manipulate data at the Oracle Database block level. No need to say that it’s very powerful and extremely dangerous because you can corrupt data/header blocks. There’s an unofficial but very comprehensive manual for BBED. It’s written by Graham Thornton. You can download it as PDF: http://orafaq.com/papers/dissassembling_the_data_block.pdf

The object code of BBED is shipped for earlier releases of Oracle. All you need is to compile it. On Oracle 11g, the required files are not shipped. So you need to copy the following files from an Oracle 10g home to Oracle 11g home:
````
$ORACLE_HOME/rdbms/lib/sbbdpt.o
$ORACLE_HOME/rdbms/lib/ssbbded.o
$ORACLE_HOME/rdbms/mesg/bbedus.msb
$ORACLE_HOME/rdbms/mesg/bbedus.msg
````

<!--more-->

What will you do if you don’t have access to any Oracle 10g software home? As you know, Oracle doesn’t provide a link to download Oracle 10g anymore. You may open a service request and ask for it, but there’s an easier way: You can get the required files by downloading the 10.2.0.5 patchset from My Oracle Support. Download p8202632_10205_Linux-x86-64.zip, and then issue the following commands (I assume that you have already set the Oracle environment variables):

````
unzip -j p8202632_10205_Linux-x86-64.zip */oracle.rdbms/10.2.0.5.0/1/DataFiles/filegroup48.1.1.jar -d /tmp

unzip -j p8202632_10205_Linux-x86-64.zip */oracle.rdbms.util/10.2.0.5.0/1/DataFiles/filegroup6.1.1.jar -d /tmp

unzip -j /tmp/filegroup48.1.1.jar sbbdpt.o ssbbded.o -d /tmp

unzip -j /tmp/filegroup6.1.1.jar bbedus.ms* -d /tmp

cp /tmp/s*bd*.o $ORACLE_HOME/rdbms/lib

cp /tmp/bbedus.ms* $ORACLE_HOME/rdbms/mesg
````

When the files are copied, you can compile BBED:

````
make -f $ORACLE_HOME/rdbms/lib/ins_rdbms.mk BBED=$ORACLE_HOME/bin/bbed $ORACLE_HOME/bin/bbed
````

BBED tool will ask you password when you try to run it. It’s not hard to find if you can use GNU debugger. You can even find it if you examine the strings in the file. Here is the password: BLOCKEDIT

Be sure to read Graham Thornton’s great manual, and be careful when playing with BBED!
