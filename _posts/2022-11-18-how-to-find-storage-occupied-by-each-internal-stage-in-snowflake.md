---
title: "How To Find Storage Occupied by Each Internal Stage in Snowflake?"
layout: post
---

The account usage view has two views related to stages: STAGES and STAGE_STORAGE_USAGE_HISTORY. The STAGES view helps list all the stages defined in your account but does not show how much storage each stage consumes. The STAGE_STORAGE_USAGE_HISTORY view shows the total usage of all stages but doesn't show detailed use. 

I wrote the following script to list the internal stages (and their occupied storage) in the active database:

{% gist abbc604b0e69ff0c545d014c167b24ba %}

<!--more-->

Here is the step-by-step explanation of the above script:

* Line 1: Decleration of variables
* Line 2: A resultset is a SQL data type that points to the result set of a query.
* Line 3: A variant variable to hold the list of stages and sizes
* Lines 5: Total_size will store the size of a stage
* Lines 6: This query will convert the variant data to a table with two columns (stage_name and total_bytes)
* Lines 7-10) Defining a cursor to list all internal stages
* Line 11) Begin of the anonymous code block
* Line 12) Initializing rpt variable as an empty array
* Line 13) A loop for each record in cursor (each stage in the database)
* Line 14) Begining of the block that we handle exceptions
* Line 15) The name of the stagen is assigned to the name variable 
* Line 16) Execute the LS command for the stage and store the result in the RES variable
* Line 17) Open a new cursor to process the rows in the RES variable
* Line 18) Total size is 0
* Lines 19-21) The size of each file in the stage, is added to the total_size variable
* Line 22) Add the stage name and the total_size as a new element to the RPT array
* Line 23-25) Exception handling section to return -1 as the size if we can't access the stage
* Line 26) End ot the block that we handle exceptions
* Line 27) Repeat this process for each stage. The loops was stated on line 12
* Line 28) Convert the array to a table with two columns (stage_name and total_bytes)
* Line 29) Return the result as a table
* Line 30) Ends the anonymous block
