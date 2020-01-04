+++
title = "Use CachedRowSet and get rid of low level SQL"
date = 2013-12-01T12:00:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "sql", "jdbc"]
+++
The interface CachedRowSet provides interesting mechanisms for one willing to work with JDBC with ease. A CachedRowSet is basically a matrix in memory which maps to objects in your database, like this:

* you have as much columns as fields in a given database table
* you have as much rows as number of records in given a database table
* the idea applies to views and joined tables as well

Instead of sending SQL commands to the database, you change cells in this matrix and, when you have finished all your changes in memory, you ask the CachedRowSet to do the dirty work for you. Example:

```java
    // update current row
    crs.updateShort("Age", 58);
    crs.updateInt("Salary", 150000);
    crs.updateRow();

    // insert new row
    crs.moveToInsertRow();
    crs.updateString("Name", "John Smith");
    crs.updateInt("Age", 42);
    crs.updateShort("Salary", 78000);
    crs.insertRow();
    crs.moveToCurrentRow();

    // update the underlying database
    crs.acceptChanges();
```

The CachedRowSet implementation is a refined piece of software which takes care of all details related to JDBC programming. You not only have something more convenient than coding SQL commands by hand, but you also have stable software at your service doing quality work in the right way it should be done.


In addition, a CachedRowSet is meant to be used disconnected from the database most of the time. After you load a CacheRowSet with data from the database, you can serialize it, send to the web browser, get it back modified and only after that you call acceptChanges. You can also build a CachedRowSet from scratch, populate it by hand and call acceptChanges.

More info: [CachedRowSet Javadoc API](http://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/CachedRowSet.html)


## CachedRowSet on Java7

Java7 ships with a RI (reference implementation) of interface CachedRowSet. Just use it as you would with any other class or interface. Have fun!

## CachedRowSet on Java5 and Java6

There are only interfaces as part of the JDK. You will have to download the RI (reference implementation) by hand. You can download it from here:

* [JDBC Rowset 1.0.1 JWSDP 1.4 Co-Bundle MREL](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-database-419422.html#jdbc_rowset_tiger-jwsdp-1_0_1-1_4-mrel-oth-JPR) (sorry, this link is broken)

Uncompress and upload rowset.jar into your artifact repository emplyoing this (g)roup/(a)pplicaiton/(v)ersion: ``javax.sql.rowset:rowset:1.0.1``

### Documentation

```bash
#/bin/bash

ls jdbc_rowset_tiger-1_0_1-mrel-jwsdp.zip

unzip jdbc_rowset_tiger-1_0_1-mrel-jwsdp.zip

ls jdbc-rowset/lib/rowset.jar
firefox jdbc-rowset/docs/index.html
```

Note: You can find "rowset" in Maven repositories around, but [that dependency will not help you](https://mvnrepository.com/artifact/javax.sql/rowset/1.0.1) since it refers to a pom.xml file, not the rowset.jar file you are interested. There's no rowset.jar file stored in Maven Central due to licensing issues.

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
