---
layout: post
title:  "Reverse Engineering a Proprietary DB using Qlikview (pt 1)"
date:   2016-01-20 00:00:00 
categories: database qlikview reverse-engineer
---
How to go about extracting data from a database that you know nothing about, no ER Diagram, no SME just a computer and connection string?


##Download Qlikview
The first thing to do is to <a href="http://qlikview.com" target="_blank">download and install Qlikview</a>.  You can get a free personal license to use that doesn't expire.

+	Create a new QV file
Create a new document and use the username and password to create a new connection to the database.  Adding the tables to the script add `QUALIFY *;` to the top.
+	Add All the Tables to the Script
This is a very important step.  In the future we are actually going to look for specific data members so the more data we have to sift through, the likelier we will find the correct column and table.
+	Add the Search Object to the dashboard
The search object will not only look for columns that match the search criteria, but also search the data within those columns that match the search criteria.
+	Start searching for criteria and note the tables and columns that contain possible matches


##Explanation 
By default Qlikview joins on columns that have the same name.  `QUALIFY *` prepends the table name to each column to ensure that each column name is unique.  Consequently, there will be no related data outside of the data on each table because none of the tables are joined.  When dealing with high volumes of data it is advantageous to have as little processing as possible given the limited resources of a laptop or desktop.  When trying to reverse engineer a database the first step is to identify columns that can be a starting point.