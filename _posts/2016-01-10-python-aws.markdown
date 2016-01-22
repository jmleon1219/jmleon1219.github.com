---
layout: post
title:  "Faster, More Comprehensive Testing using Python"
date:   2016-01-10 00:00:00 
categories: python aws
---
Recently, I was given the task to do some integration testing around a process that has files originate in Amazon Web Services S3 buckets.  Then reconsile those files with csv files that are output after a warehousing process.  On the surface this may seem like a daunting task, matching files that cotain tens of thousands of rows line by line.  Luckily we have Python at our disposal! ([The complete code can be found on my bitbucket account](https://bitbucket.org/jacob_leon/integration-test))

##Python
The great thing about Python is that the language has an extensive list of libraries.  The library that can interact with AWS is called Boto3 (`pip install boto3`).  Boto3 is not limited to S3, there is also functionalilty to manage EC2 instances.

####Downloading from AWS S3
{% highlight python %}
	def __downloadFromS3(bucket_name,dest_path):
		csv_data = list()
		s3 = boto3.resource('s3')
		s3_file_path = dest_path
		for mybucket in s3.buckets.all():
			if(mybucket.name == bucket_name):
				maxdate = self.__findLatestSnapshotFile([str(object.key) for object in mybucket.objects.all() if match('.*\.csv',str(object.key))])
				for object in mybucket.objects.all():
					if(match('.*{0}\.csv'.format(maxdate),str(object.key))):
						filename =object.key.split('/')[-1]
						contents = object.get()
						with open(s3_file_path + filename,'wb') as file:
							txt = contents['Body'].read()
							file.write(txt)
							file.close
{% endhighlight %}

The above code logs into the S3 instance and looks through all the buckets for one called reports.  Once reports is found, everything with .csv is download locally to the path defined in the "s3_file_path" variable.


####Reading JSON doc
{% highlight python %}

	def readJSONSourceFiles(self,source_path,dest_path):
		with open(source_path, encoding='utf-8') as json_file:
			self.source_data = json.loads(json_file.read())['results']['details']

{% endhighlight %}

This block is pretty self explanatory.  It reads in the JSON file at the specified by the source_path variable and reads the file into memory.

####Comparing the Two Sources
{% highlight python %}

	def __compareTwoLists(self,list_a, list_b, mapping_recs_filter):
		match_lst = []
		non_match_lst = []	

		for j in list_a:
			for r in list_b:
				if(mapping_recs_filter(j,r)):
					match_lst.append(j)
					break
			else:
				non_match_lst.append(j)
				
		return match_lst, non_match_lst

{% endhighlight %}

This code simply compares the items in the two lists.  Once it finds a match stops (or breaks) from trying to make another match and moves on to the next record.

###Rest of the Code
This was just a snippet of what I actually created to do integration testing.  I created an entire class that can accomodate most requirements.  The rest of the code can be found at [my bitbucket account](https://bitbucket.org/jacob_leon/integration-test).