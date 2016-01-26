---
layout: post
title:  "Reverse Engineering a Proprietary DB using Qlikview (pt 2)"
date:   2016-01-22 00:00:00 
categories: database qlikview reverse-engineer
---
Now that some of the data members turned up in one or more columns, the task is to join the table to your main query or a driving table.  This can be much more difficult if the database is using non standard functionality and is using a series of tables to show "flexible fields".

##Analyze Other Columns
The first step is to look at the other columns on the table where the relevant data is found.  Pay attention to anything with "id" in the name.  There is a good chance that an "id" column will be referenced by another table as a foreign key.  Plug the full column name into Qlikview's search to find a list of tables with the "id" column. 

##Process to Put Tables Together
Start on the table where the data was found:

1.  Look at the other columns in the table.  Focus on the ones that have "id" in the name, they are the best bet to be used as foreign keys in another table.
2.  Plug the columns that have the best chance of being foreign keys (start with, but not limited to "id"s) into Qlikview's universal search.  
3.  Focus on the tables with more meaningful names.  These tables are likelier to be part of your main query.  If a query isn't developed yet, a table with a more meaningful name is a better bet to be your driving table.
4.  Join the two tables together via common column.

_*Change focus to the new table and repeat this process until a table with sought-after data is found_


##Caveats
-  When joining to another table, take note of the cardinality.  There are times when an "id" column is not a primary key by itself, but is part of a larger composite key.  This is more of an issue when dealing with "flexible fields" because there can be another column that describes the value column.
-  Data in flex fields may need additional translation.  For instance, the application might have "True" or "Yes" in the front end but the value in the database might be 1 or 0.