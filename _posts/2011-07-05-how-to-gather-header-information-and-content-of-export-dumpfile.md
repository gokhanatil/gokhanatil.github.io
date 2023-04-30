---
title: "How To Gather The Header Information And The Content Of An Export Dumpfile?"
layout: post
---

I’ve found a great document at My Oracle Support (Metalink): How to Gather the Header Information and the Content of an Export Dumpfile? [ID 462488.1]. The document explains how to extract DDL statements from dump files and how to use the DBMS_DATAPUMP.GET_DUMPFILE_INFO procedure to gather header information for both original and data pump exports.

In summary,

The Data Definition Language (DDL) statements in a DataPump dump file can be extracted with the parameter SQLFILE:
````
impdp DIRECTORY=testdir DUMPFILE=test.dmp SQLFILE=ddlcommands.sql FULL=y
````

Note: This command will not import the data, but it still needs a valid database connection.

The Data Definition Language (DDL) statements in a original export file can be extracted with the parameter SHOW:
````
imp FILE=test.dmp LOG=test.log FULL=y SHOW=y
````

The sample PL/SQL script can be found at MOS note. Unfortunately, it can only be used in an Oracle10g Release 2 or any higher release database.

After I’ve made some tests with PL/SQL code, I decided to examine Oracle export files and write my own utility to read the header information.

<!--more-->

So I wrote a simple Perl script (dumpinfo.pl). You can get the full source code from GitHub:

[![Download](/assets/gitHub-download-button.png)](https://github.com/gokhanatil/oracleDBAscripts/blob/master/dumpinfo.pl)

Sample outputs:
````
$ ./dumpinfo.pl testdir/gokhan.dmp

dumpinfo (C) 2011 Gokhan Atil http://www.gokhanatil.com

 ........Filetype = Datapump dumpfile
 ......DB Version = 10.02.00.03.0
 File Version Str = 1.1
 File Version Num = 257
 ........Job Guid = a73d207ae6b75ceee04400144fad4c76
 Master Table Pos = 21
 Master Table Len = 144144
 ......Charset ID = 39
 ...Creation date = 7-4-2011 13:50:33
 ........Job Name = "GOKHAN"."SYS_EXPORT_SCHEMA_01"
 ........Platform = SVR4-be-64bit-8.1.0
 ........Language = WE8ISO8859P9
 .......Blocksize = 4096

$ ./dumpinfo.pl testdir/test.dmp

dumpinfo (C) 2011 Gokhan Atil http://www.gokhanatil.com

 ........Filetype = Classic Export file
 ..Export Version = 10.02.01
 .....Direct Path = 1 (Direct Path)
 ...Creation date = Tue Jul 5 10:53:29 2011
 ````
 
Please send me your feedback (and bug reports).

Note: I rewrote the same script in [Python](https://github.com/gokhanatil/oracleDBAscripts/blob/master/dumpinfo.py)!
