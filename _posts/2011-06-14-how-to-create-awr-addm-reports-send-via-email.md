---
title: "How to Create AWR And ADDM Reports And Send Them Via Email?"
layout: post
---

I've seen a question on the OTN forum about how to create a job in Grid Control for generating AWR/ADDM reports and sending these reports via email. As I know OEM Grid Control doesn’t have such a job template but we can write a PL/SQL script for this task and define it as a job, so we can automate it for all databases.

First, let’s check how we can generate AWR reports. To be able to get AWR reports in plain text, we can use:

Syntax (for Oracle 10.2):

```sql
DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_TEXT(
l_dbid IN NUMBER,
l_inst_num IN NUMBER,
l_bid IN NUMBER,
l_eid IN NUMBER,
l_options IN NUMBER DEFAULT 0)
RETURN awrrpt_text_type_table PIPELINED;
```

If we want the report in HTML, we can use:

```sql
DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_HTML(
l_dbid IN NUMBER,
l_inst_num IN NUMBER,
l_bid IN NUMBER,
l_eid IN NUMBER,
l_options IN NUMBER DEFAULT 0)
RETURN awrrpt_text_type_table PIPELINED;
```

<!--more-->

As you see both functions accept the same parameters, the only difference is the format of the report. The parameters of these functions:

l_dbid: The database identifier
l_insT_num: The instance number
l_bid: The 'Begin Snapshot' ID
l_eid: The 'End Snapshot' ID
l_options: A flag to specify to control the output of the report. The default value is 0, if we set it to 8, the report will include the ADDM-specific sections including the Buffer Pool Advice, Shared Pool Advice, and PGA Target Advice

One of the important points is, we need to determine the database ID, instance number, and snapshots dynamically to be able to define it as a job in OEM Grid Control. I query DBA_HIST_SNAPSHOT to find the beginning and end snapshot IDs according to variables starttime and endtime. I read the database ID and instance number from GV$DATABASE.

Here’s a simple PL/SQL script to create an **AWR report** in HTML and mail it:

{% gist eb4b4ea01771ca350122fc84d7dde778 %}

To be able to create ADDM Reports, we can use the DBMS_ADVISOR package:

```plsql
DBMS_ADVISOR.CREATE_TASK('ADDM',tid,tname,'Name of the ADDM Report');
DBMS_ADVISOR.SET_TASK_PARAMETER( tname,'START_SNAPSHOT',bid );
DBMS_ADVISOR.SET_TASK_PARAMETER( tname,'END_SNAPSHOT',eid );
DBMS_ADVISOR.EXECUTE_TASK( tname );
```

ADDM reports are created as a background jobs, so we wait until they’re completed and get the report:

```plsql
SELECT DBMS_ADVISOR.GET_TASK_REPORT(tname) FROM DUAL;
```

Here’s the simple PL/SQL block to create **ADDM report** and mail it:

{% gist b7c5af041e1ffa6ec7df71a45cc22fdb %}

I tested these scripts on 10.2 and 11.2, and they worked fine. By the way, don’t forget to set ACL if you’ll run them in 11g+:

```sql
BEGIN
  DBMS_NETWORK_ACL_ADMIN.create_acl (
    acl          => 'smtpserver.xml', 
    description  => 'Connection to SMTP',
    principal    => 'GOKHAN',    
    is_grant     => TRUE, 
    privilege    => 'connect',
    start_date   => SYSTIMESTAMP,
    end_date     => NULL);
  COMMIT;
END;
/

BEGIN
  DBMS_NETWORK_ACL_ADMIN.assign_acl ( acl => 'smtpserver.xml',  
  host => 'oursmtpserver', lower_port  => 25,  upper_port  => 25 ); 
  COMMIT;
END;
/
```

I wrote these scripts only for demonstration purposes. They are not optimized and have almost no error checking, so be careful when using them in production environments.

Anyway, our scripts are ready for testing. We can create the job in Grid Control: 

* Login to OEM Grid Control
* Click on the Jobs tab 
* Choose SQL Script as the job type and click on the "Go" button
* Enter a name and description of the new job 
* Click Add to add the target databases. 
* Click on the "Parameters" tab
* Copy/paste one of the above scripts into the textbox labeled "SQL Script"
* Set the credentials and schedule the job as you wish
* Click the submit button to create the job
