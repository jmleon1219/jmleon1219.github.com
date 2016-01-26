---
layout: post
title:  "Reverse Engineering Rational Team Concert (RTC)"
date:   2016-01-22 00:00:00 
categories: database qlikview reverse-engineer rtc ibm
---
This post is the real world application of my previous two posts.  The database that we are going to reverse engineer is IBM's Rational Team Concert.  Rational Team Concert is a client-server collaboration tool that is used to track software development tasks.  This example has two main types of entries, Issues and Change Requests.

##Initial Query
The task is to extract the data from the database and match what the users see on the front end.  When looking at the tables it was clearly evident that the REQUEST table was the main driver of all data in the solution.  It contains many foreign keys and it is a very wide table.  


{% highlight sql%}
	SELECT *
	  FROM RTC_DWH_PROD.RIODS.REQUEST R
INNER JOIN RTC_DWH_PROD.RIODS.PROJECT P
		ON R.PROJECT_ID = P.PROJECT_ID
INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_TYPE RT
		ON R.REQUEST_TYPE_ID = RT.REQUEST_TYPE_ID
INNER JOIN RTC_DWH_PROD.RIODS.TEAM TM
		ON R.TEAM_ID = TM.TEAM_ID
INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_CATEGORY RCAT
		ON R.REQUEST_CATEGORY_ID = RCAT.REQUEST_CATEGORY_ID
INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_SEVERITY REQSEV
		ON R.REQUEST_SEVERITY_ID = REQSEV.REQUEST_SEVERITY_ID
INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_PRIORITY REQPR
		ON R.REQUEST_PRIORITY_ID = REQPR.REQUEST_PRIORITY_ID
INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_STATE REQST
		ON R.REQUEST_STATE_ID = REQST.REQUEST_STATE_ID
INNER JOIN RIODS.RESOURCE OWNR
		ON R.OWNER_ID = OWNR.RESOURCE_ID		
	 WHERE P.NAME = 'MyProject';

{% endhighlight %}

Anyone with a SQL background would find it fairly easy to join the rest of the tables to the REQUEST table.  Each row is a separate entry on the application front end and each additional tables gives the request row more context.

##Missing Columns?
The initial query returns most of the desired data, but there are some glaring omissions such as:

-  Category
-  Sub-Category
-  Program
-  Project
-  Resolution Notes
-  Escalated Flag

It is evident that some functionality is re-purposed for other data.  For instance the "REQUEST_CATEGORY" table was used to hold a field that was called "Filed Against".  The most logical conclusion is that some non-standard functionality was being used.

##Finding the Missing Columns
Some data members for the missing column can be gathered by looking through the front end. However, first review [Reverse engineering a DB part 1.](/database/qlikview/reverse-engineer/2016/01/19/reverse-engineer-db-pt1.html) That post goes into the process of downloading qlikview and setting up the application to be able to search large amounts of data very quickly.
<br>
<br>
Next take some of these data members and plug them into the search object in qlikview.  Matching entries and corresponding columns should be highlighted.  Once the column containing the data is found the trick is to tie it back to your main query.  The process is detailed in my post [Reverse Engineering a DB part 2](/database/qlikview/reverse-engineer/2016/01/21/reverse-engineer-db-pt2.html).  Essentially, a key on the table containing the sought after data needs to be identified that is used as a foreign key to another table.  Once a corresponding table is found, the focus shifts to this new table and the process repeats.  The process ends once the two queries can tie themselves together. 

{%  highlight sql %}
    Select REQSE.REQUEST_ID,
		   RENUM.PROJECT_ID,
		   RENUM.LITERAL_NAME as PROGRAM_NAME
	  from RICALM.REQUEST_ENUMERATION RENUM
INNER JOIN RICALM.REQUEST_STRING_EXT REQSE
		on RENUM.EXTERNAL_ID = REQSE.VAL
	 WHERE REQSE.NAME = 'program';

{% endhighlight %}
######_Example of a query that was developed to accommodate non-standard functionality_

##Final Query
Using the process that was outlined in this post, all of the columns were found and added to the initial query yielding:

{% highlight sql %}
	 SELECT R.EXTERNAL_KEY1 as Id,
			P.NAME as "Module",
			RCAT.NAME as "Filed Against",
			prog.PROGRAM_NAME as Program,
			prj.PROJECT_NAME as Project,
			cat.CATEGORY_NAME as Category,
			subcat.SUBCATEGORY_NAME as "Sub Category",
			R.NAME as Summary,
			RT.NAME as Type,
			R.CREATION_DATE as "Creation Date",
			R.RESOLVED_DATE as "Resolution Date",
			REQSEV.NAME as Severity,
			REQPR.NAME as Priority,
			REQST.NAME as Status,
			TGTRL.VAL as "Target Release",
			RESDETS.VAL as "Resolution Notes",
			OWNR.FULL_NAME as Owner,
			CASE ESCAL.VAL
				WHEN 1 THEN 'True'
				ELSE 'False'
			 END as Escalated
	   FROM RTC_DWH_PROD.RIODS.REQUEST R
 INNER JOIN RTC_DWH_PROD.RIODS.PROJECT P
		 ON R.PROJECT_ID = P.PROJECT_ID
 INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_TYPE RT
		 ON R.REQUEST_TYPE_ID = RT.REQUEST_TYPE_ID
 INNER JOIN RTC_DWH_PROD.RIODS.TEAM TM
		 ON R.TEAM_ID = TM.TEAM_ID
 INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_CATEGORY RCAT
		 ON R.REQUEST_CATEGORY_ID = RCAT.REQUEST_CATEGORY_ID
 INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_SEVERITY REQSEV
		 ON R.REQUEST_SEVERITY_ID = REQSEV.REQUEST_SEVERITY_ID
 INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_PRIORITY REQPR
		 ON R.REQUEST_PRIORITY_ID = REQPR.REQUEST_PRIORITY_ID
 INNER JOIN RTC_DWH_PROD.RIODS.REQUEST_STATE REQST
		 ON R.REQUEST_STATE_ID = REQST.REQUEST_STATE_ID
 INNER JOIN RIODS.RESOURCE OWNR
		 ON R.OWNER_ID = OWNR.RESOURCE_ID
  LEFT JOIN RICALM.REQUEST_STRING_M_EXT TGTRL
		 ON R.REQUEST_ID = TGTRL.REQUEST_ID
		AND TGTRL.NAME = 'target_release'
  LEFT JOIN RICALM.REQUEST_STRING_L_EXT RESDETS
		 ON R.REQUEST_ID = RESDETS.REQUEST_ID
		AND RESDETS.NAME = 'resolution_notes'
  LEFT JOIN RICALM.REQUEST_BOOL_EXT ESCAL
		 ON R.REQUEST_ID = ESCAL.REQUEST_ID
		AND ESCAL.NAME = 'escalated'
  LEFT JOIN ( Select REQSE.REQUEST_ID,
					 RENUM.PROJECT_ID,
					 RENUM.LITERAL_NAME as PROJECT_NAME
				from RICALM.REQUEST_ENUMERATION RENUM
		  INNER JOIN RICALM.REQUEST_STRING_EXT REQSE
				  ON RENUM.EXTERNAL_ID = REQSE.VAL
			   WHERE 1=1
				 AND REQSE.NAME = 'project_list') prj
		 ON R.REQUEST_ID = prj.REQUEST_ID
	    AND P.PROJECT_ID = prj.PROJECT_ID
  LEFT JOIN ( Select REQSE.REQUEST_ID,
					 RENUM.PROJECT_ID,
					 RENUM.LITERAL_NAME as PROGRAM_NAME
				from RICALM.REQUEST_ENUMERATION RENUM
		  INNER JOIN RICALM.REQUEST_STRING_EXT REQSE
				  ON RENUM.EXTERNAL_ID = REQSE.VAL
			   WHERE 1=1
				 AND REQSE.NAME = 'program_attrib') prog
	    ON R.REQUEST_ID = prog.REQUEST_ID
	   AND P.PROJECT_ID = prog.PROJECT_ID
 LEFT JOIN ( Select REQSE.REQUEST_ID,
					RENUM.PROJECT_ID,
					RENUM.LITERAL_NAME as CATEGORY_NAME
			   from RICALM.REQUEST_ENUMERATION RENUM
		 INNER JOIN RICALM.REQUEST_STRING_EXT REQSE
				 on RENUM.EXTERNAL_ID = REQSE.VAL
			  WHERE 1=1
				AND REQSE.NAME = 'category') cat
	   ON R.REQUEST_ID = cat.REQUEST_ID
	  AND P.PROJECT_ID = cat.PROJECT_ID
LEFT JOIN ( Select REQSE.REQUEST_ID,
				   RENUM.PROJECT_ID,
				   RENUM.LITERAL_NAME as SUBCATEGORY_NAME
			  from RICALM.REQUEST_ENUMERATION RENUM
		INNER JOIN RICALM.REQUEST_STRING_EXT REQSE
				on RENUM.EXTERNAL_ID = REQSE.VAL
			 WHERE 1=1
			   AND REQSE.NAME = 'subcategory') subcat
	   ON R.REQUEST_ID = subcat.REQUEST_ID
	  AND P.PROJECT_ID = subcat.PROJECT_ID
WHERE P.NAME = 'MyProject'

{% endhighlight %}