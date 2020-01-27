# Region Size Calculator

## Prerequisite

1.Gemfire 9.x

2.Registered Account of "Pivotal Commercial Maven Repository" 

3.Apache Maven installed such as 3.6.1 



## Function Execution Usage

This Function Execution Service： `region-size-calculator` Usage

**Input:**

- *Required argument*:  the name of the region
- *Optional argument*: the number of samples to take. If you have a region with 1 billion entries, you may deem it unnecessary to go through each entry and calculate its size. For this reason, this argument will limit the number of entries to sample and the total size will be projected from the results * the number of entries in the region.
- Function execution arguments in gfsh are **comma-delimited strings**


**Output:**

Output items are as the below, the size unit is **Byte**.
- *Deserialized values size* 
- *Serialized values size*
- *Keys size*
- *Region type*
- *Entries*

**Current total Region size = Deserialized values size + Serialized values size + Keys size**

*For example:*
```
gfsh>execute function --id=region-size-calculator --arguments="exampleRegion,10" --member=server1
Execution summary

         Member ID/Name          | Function Execution Result
-------------------------------- | ----------------------------------------------------------------------------------------------------------------
172.17.0.2(server1:162)<v1>:1025 | [{Deserialized values size=720, Serialized values size=170, Keys size=480, Region type=Partitioned, Entries=10}]
```

## Considerations

The cost of calculating object sizes in a high-speed data grid is too expensive to perform on each entry. This Region Size Calculator calculates the sizes based on the following a few considerations.

1.Understanding GemFire Serialization

GemFire stores data in serialized form. It will store the object in deserialized form in some circumstances temporarily. This deserialized object will later be garbage collected. Therefore, the actual region size will flux depending on your operations.
If you store your objects using PDX and do queries with “Select *”, GemFire will store the object in deserialized form until the next GC. If your queries use fieldnames, such as “Select lastName, firstName”, GemFire will maintain the object in serialized form.
Function execution will also affect PDX deserialization. If your function casts a PDX object to it’s domain object, the object will be stored in deserialized form on that node and that node only temporarily. 

2.The Region Size Calculator will return both the size of the deserialized storage and serialized storage. You can estimate the real size of the region based on your use. If you do not use “Select *” and do not cast PDX objects to the Domain object in functions, your region size will be the sum of the keys and the deserialized values.


## Installation

**_Step1:_** 

Download this project to a local env and then unzip it as a folder such as [geode-region-sizer/functions].

**_Step2:_**

Config the .m2/settings.xml with your registered account of "Pivotal Commercial Maven Repository" by referring the below document.
https://gemfire.docs.pivotal.io/99/gemfire/getting_started/installation/obtain_gemfire_maven.html

```
<settings>
       <servers>
           <server>
               <id>gemfire-release-repo</id>
               <username>MY-USERNAME@example.com</username>
               <password>MY-DECRYPTED-PASSWORD</password>
           </server>
       </servers>
   </settings>
 ```

**_Step3:_** 

Use mvn compile/mvn package command or IDE such as Visual Studio Code build the project, then you will find a functions-0.0.1-SNAPSHOT.jar from target folder.
For example:

```
XXXXX_MBP:functions jaxu$ mvn clean package
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------< io.pivotal:functions >------------------------
[INFO] Building functions 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
......
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ functions ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/jaxu/Downloads/geode-region-sizer/functions/target/classes
[INFO] /Users/xxxxx/Downloads/geode-region-sizer/functions/src/main/java/io/pivotal/utils/SizeCalculator.java: Some input files use or override a deprecated API.
[INFO] /Users/xxxxx/Downloads/geode-region-sizer/functions/src/main/java/io/pivotal/utils/SizeCalculator.java: Recompile with -Xlint:deprecation for details.
[INFO] /Users/xxxxx/Downloads/geode-region-sizer/functions/src/main/java/io/pivotal/functions/SizeCalculatorFunction.java: /Users/jaxu/Downloads/geode-region-sizer/functions/src/main/java/io/pivotal/functions/SizeCalculatorFunction.java uses unchecked or unsafe operations.
[INFO] /Users/xxxxx/Downloads/geode-region-sizer/functions/src/main/java/io/pivotal/functions/SizeCalculatorFunction.java: Recompile with -Xlint:unchecked for details.
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ functions ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/jaxu/Downloads/geode-region-sizer/functions/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ functions ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ functions ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ functions ---
[INFO] Building jar: /Users/xxxxx/Downloads/geode-region-sizer/functions/target/functions-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.187 s
[INFO] Finished at: 2020-01-27T18:03:04+09:00
[INFO] ------------------------------------------------------------------------
```


**_Step4:_** 

You can deploy the functions-0.0.1-SNAPSHOT.jar into a gemfire cluster by gfsh command.

For example:

```
gfsh>deploy --jar=/Users/xxxxx/Downloads/geode-region-sizer/functions/target/functions-0.0.1-SNAPSHOT.jar

Deploying files: functions-0.0.1-SNAPSHOT.jar
Total file size is: 0.01MB

Continue?  (Y/n): Y
      Member        |         Deployed JAR         | Deployed JAR Location
------------------- | ---------------------------- | ------------------------------------------------------------------------------------
mix-delightful-ball | functions-0.0.1-SNAPSHOT.jar | /Users/xxxxx/Downloads/vsc_work/clusters/cacheserver1/functions-0.0.1-SNAPSHOT.v2.jar

gfsh>list deployed
      Member        |             JAR              | JAR Location
------------------- | ---------------------------- | ------------------------------------------------------------------------------------
mix-delightful-ball | functions-0.0.1-SNAPSHOT.jar | /Users/xxxxx/Downloads/work/clusters/cacheserver1/functions-0.0.1-SNAPSHOT.v2.jar
```


## Reference

1.Gemfire8.2.x based gemfire region size calculator utility.

https://github.com/Pivotal-Data-Engineering/gemfire-region-size-calculator

2.Gemfire6.x based gemfire region calculator utility.

https://communities.vmware.com/docs/DOC-20695
