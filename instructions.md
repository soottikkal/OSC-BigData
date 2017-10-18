#Instructions for the workshop

#connect to owens
   ssh username@owens.osc.edu

#copy files
    cp -r ~soottikkal/workshop/Oct17-Bigdata ./

#check files
    cd Oct17-Bigdata
ls

request 1 interactive node
qsub -I -l nodes=1:ppn=28 -l walltime=04:00:00 -A PZS0687

#launch spark
module load spark/2.0.0
pyspark --executor-memory 4G --driver-memory 4G

#Example 1: Unstructured data

#create a RDD
#make sure that the path is correct

data = sc.textFile(“Oct17-Bigdata/spark/README.md”)
#count number of lines

data.count()
99
#see the content of the RDD

data.take(3)
[u’# Apache Spark’, u’’, u’Spark is a fast and general cluster computing system for Big Data. It provides’]
data.collect()
#check data type

type(data)
<class ‘pyspark.rdd.RDD’>
#transformation of RDD

linesWithSpark = data.filter(lambda line: “Spark” in line)
#action on RDD
linesWithSpark.count()
19
#combining transformation and actions

data.filter(lambda line: “Spark” in line).count()
19
#wordcount

wordCounts = data.flatMap(lambda line: line.split()).map(lambda word: (word,1)).reduceByKey(lambda a, b: a+b)
wordCounts.collect()
#Example 2: Structured data

#About the data: http://kdd.ics.uci.edu/databases/kddcup99/kddcup99

#load data and run basic operations
#make sure that the path is correct
#ignore warnings

data=spark.read.csv(“Oct17-Bigdata/spark/data.csv”, header=‘TRUE’)
data.count()
494021
data.take(1)
[Row(dst_bytes=u’5450’, duration=u’0’, flag=u’SF’, protocal_type=u’tcp’, service=u’http’, src_bytes=u’181’)]
data.take(3)
[Row(dst_bytes=u’5450’, duration=u’0’, flag=u’SF’, protocal_type=u’tcp’, service=u’http’, src_bytes=u’181’), Row(dst_bytes=u’486’, duration=u’0’, flag=u’SF’, protocal_type=u’tcp’, service=u’http’, src_bytes=u’239’), Row(dst_bytes=u’1337’, duration=u’0’, flag=u’SF’, protocal_type=u’tcp’, service=u’http’, src_bytes=u’235’)]
data.printSchema()
root
|-- dst_bytes: long (nullable = true)
|-- duration: long (nullable = true)
|-- flag: string (nullable = true)
|-- protocal_type: string (nullable = true)
|-- service: string (nullable = true)
|-- src_bytes: long (nullable = true)
data.show(5)
±--------±-------±—±------------±------±--------+
|dst_bytes|duration|flag|protocal_type|service|src_bytes|
±--------±-------±—±------------±------±--------+
| 5450| 0| SF| tcp| http| 181|
| 486| 0| SF| tcp| http| 239|
| 1337| 0| SF| tcp| http| 235|
| 1337| 0| SF| tcp| http| 219|
| 2032| 0| SF| tcp| http| 217|
±--------±-------±—±------------±------±--------+
only showing top 5 rows
data.select(“dst_bytes”,“flag”).show(5)
±--------±—+
|dst_bytes|flag|
±--------±—+
| 5450| SF|
| 486| SF|
| 1337| SF|
| 1337| SF|
| 2032| SF|
±--------±—+
only showing top 5 rows
data.filter(data.flag!=“SF”).show(5)
±--------±-------±—±------------±------±--------+
|dst_bytes|duration|flag|protocal_type|service|src_bytes|
±--------±-------±—±------------±------±--------+
| 29200| 0| S1| tcp| http| 228|
| 9156| 0| S1| tcp| http| 212|
| 0| 0| REJ| tcp| other| 0|
| 0| 0| REJ| tcp| other| 0|
| 0| 0| REJ| tcp| other| 0|
±--------±-------±—±------------±------±--------+
only showing top 5 rows
data.select(“protocal_type”, “duration”, “dst_bytes”).groupBy(“protocal_type”).count().show()
±------------±-----+
|protocal_type| count|
±------------±-----+
| tcp|190065|
| udp| 20354|
| icmp|283602|
±------------±-----+
data.select(“protocal_type”, “duration”, “dst_bytes”).filter(data.duration>1000).filter(data.dst_bytes==0).groupBy(“protocal_type”).count().show()
±------------±----+
|protocal_type|count|
±------------±----+
| tcp| 139|
±------------±----+
#SparkSQL querries
#Register data as a table named “interactions”

data.registerTempTable(“interactions”)
Select tcp network interactions with more than 1 second duration and no transfer from destination
tcp = sqlContext.sql(" SELECT duration, dst_bytes FROM interactions WHERE protocal_type =‘tcp’ AND duration>1000 AND dst_bytes=0")
tcp.show(5)
#exit from the pyspark shell
exit()

#exit form the compute node
exit

#submitting batch job

#check files
ls
cd Oct17-Bigdata
cd spark
ls
cat stati.py
cat sql.py
cat stati.pbs
cat sql.pbs

#submitg first job
qsub sql.pbs

#submit second job
qsub stati.pbs

#checking job status
qstat

make sure backtick character(Above the ‘tab’ key) entered correctly
qstat | grep whoami
ls

#check out log files after Jobs are completed
more stati.log
more sql.log

#submitting hadoop jobs
cd …/
cd hadoop
qsub sub-wordcount.pbs
qsub sub-grep.pbs

#check job status
qstat | grep whoami

#check out results after Jobs are completed
cd wordcout-out
more part-r-00000

cd …/
cd grep-out
more part-r-00000

exit

You can also:
