# Starting with MongoDB (Primeros pasos con MongoDB)

![LogoHeadMasterCES](https://sites.google.com/site/manuparra/home/logo_master_ciber.png)


[UGR](http://www.ugr.es) | [DICITS](http://dicits.ugr.es) | [SCI2S](http://sci2s.ugr.es) | [DECSAI](http://decsai.ugr.es)

Manuel J. Parra Royón (manuelparra@decsai.ugr.es) & José. M. Benítez Sánchez (j.m.benitez@decsai.ugr.es)


Table of Contents
=================
   
   * [Using MongoDB, why? where? what? (Theory)](#using-mongodb-why-where-what)
      * [Documents instead row/cols](#documents-instead-rowcols)
      * [Documents datatypes](#documents-datatypes)
   * [Starting with MongoDB (Practice)](#starting-with-mongodb)
      * [Connecting with MongoDB Service:](#connecting-with-mongodb-service)
      * [Selecting/Creating/Deleting DataBase](#selectingcreatingdeleting-database)
      * [Selecting/Querying/Filtering](#selectingqueryingfiltering)
      * [Updating documents](#updating-documents)
      * [Deleting documents](#deleting-documents)
      * [Import external data](#import-external-data)
      * [MongoDB Clients](#mongodb-clients)
   * [References](#references)

# Using MongoDB, why? where? what?

MongoDB is an open-source database developed by MongoDB, Inc. 

MongoDB stores data in JSON-like documents that can vary in structure. Related information is stored together for fast query access through the MongoDB query language. MongoDB uses dynamic schemas, meaning that you can create records **without first defining the structure**, such as the fields or the types of their values. You can change the structure of records (which we call documents) simply by adding new fields or deleting existing ones. This data model give you the ability to represent hierarchical relationships, to store arrays, and other more complex structures easily. Documents in a collection need not have an identical set of fields and denormalization of data is common. MongoDB was also designed with high availability and scalability in mind, and includes out-of-the-box replication and auto-sharding.

**MongoDB main features:**

* Document Oriented Storage − Data is stored in the form of JSON style documents.
* Index on any attribute
* Replication and high availability
* Auto-sharding
* Rich queries

**Using Mongo:**

* Big Data
* Content Management and Delivery
* Mobile and Social Infrastructure
* User Data Management
* Data Hub

**Compared to MySQL:**

Many concepts in MySQL have close analogs in MongoDB. Some of the common concepts in each system:

* MySQL -> MongoDB
* Database -> Database
* Table -> Collection
* Row -> Document
* Column -> Field
* Joins -> Embedded documents, linking

**Query Language:**

From MySQL:

```
INSERT INTO users (user_id, age, status)
VALUES ('bcd001', 45, 'A');
```

To MongoDB:

```
db.users.insert({
  user_id: 'bcd001',
  age: 45,
  status: 'A'
});
```

From MySQL:

```
SELECT * FROM users
```

To MongoDB:

```
db.users.find()
```


From MySQL:

```
UPDATE users SET status = 'C'
WHERE age > 25
```

To MongoDB:

```
db.users.update(
  { age: { $gt: 25 } },
  { $set: { status: 'C' } },
  { multi: true }
)
```


## Documents instead row/cols

MongoDB stores data records as BSON documents. 

BSON is a binary representation of JSON documents, it contains more data types than JSON.

MongoDB documents are composed of field-and-value pairs and have the following structure:

```
{
   field1: value1,
   field2: value2,
   field3: value3,
   ...
   fieldN: valueN
}
```

Example of document:

```
var mydoc = {
               _id: ObjectId("5099803df3f4948bd2f98391"),
               name: 
               		{ 
               		 first: "Alan", 
               		 last: "Turing" 
               		},
               birth: new Date('Jun 23, 1912'),
               death: new Date('Jun 07, 1954'),
               contribs: [ 
               				"Turing machine", 
               				"Turing test", 
               				"Turingery" ],
               views : NumberLong(1250000)
            }
```

To specify or access a field of an document: use dot notation

```
mydoc.name.first
```

Documents allow embedded documents embedded documents embedded documents ...:

```
{
   ...
   name: { first: "Alan", last: "Turing" },
   contact: { 
   			phone: { 
   					model: { 
   						brand: "LG", 
   						screen: {'maxres': "1200x800"} 
   					},
   					type: "cell", 
   					number: "111-222-3333" } },
   ...
}
```

The maximum BSON document size is **16 megabytes!**.


### Documents datatypes

* String − This is the most commonly used datatype to store the data.
* Integer − This type is used to store a numerical value.
* Boolean − This type is used to store a boolean (true/ false) value.
* Double − This type is used to store floating point values.
* Min/ Max keys − This type is used to compare a value against the lowest and highest BSON elements.
* Arrays − This type is used to store arrays or list or multiple values into one key.
* Timestamp − ctimestamp. This can be handy for recording when a document has been modified or added.
* Object − This datatype is used for embedded documents.
* Null − This type is used to store a Null value.
* Symbol − This datatype is used identically to a string; however, it's generally reserved for languages that use a specific symbol type.
* Date − This datatype is used to store the current date or time in UNIX time format. You can specify your own date time by creating object of Date and passing day, month, year into it.
* Object ID − This datatype is used to store the document’s ID.
* Binary data − This datatype is used to store binary data.
* Code − This datatype is used to store JavaScript code into the document.
* Regular expression − This datatype is used to store regular expression.


# Starting with MongoDB

Log and connect to our system ```betatun.ugr.es``` with your credentials (as usual):

```
ssh CS<yourID>@betatun.ugr.es
```

Use your password for SSH in betatun.ugr.es

First of all, check that you have access to the mongo tools system, try this command:

```
mongo + tab
```

it will show:

```
mongo         mongodump     mongoexport   mongofiles    
mongoimport   mongooplog    mongoperf     mongorestore  mongostat     mongotop 
```

## Connecting with MongoDB Service

Write the next:

```
mongo 
```


It will connect with defaults parameters: ``localhost`` , port: ``27017`` and database: ``default``

```
MongoDB shell version: 2.6.12
connecting to: test
>
```

In order to exit: ``CTRL+C`` or ``exit``


## Selecting/Creating/Deleting DataBase

The command will create a new database if it doesn't exist, otherwise it will return the existing database.

```
> use YOURID;
```

Now you are using ``YOURID`` (``CC777777777N`` or your ID) database.

If you want to kwnow what database are you using:

```
> db
```

The ```command db.dropDatabase()``` is used to drop a existing database.

To kwnow the size of databases:

```
show dbs
```

## Creating a Collection

Basic syntax of createCollection() command is as follows:

```
db.createCollection(name, options)
```

where ``options`` specifies options about memory size and indexing.

Remember that firstly mongodb needs to kwnow what is the Database where it will create the Collection. Use ``show dbs`` and then ``use <your database>``.

```
use <YOURID>;
```

And then create the collection:

```
db.createCollection("MyFirstCollection")
```

and another more:

```
db.createCollection("MySecondCollection")
```

When created check:

```
show collections
```

In MongoDB, you don't need to create the collection. MongoDB creates collection automatically, when you insert some document (to ``MySecondCollection``):

```
db.MySecondCollection.insert({"name" : "Manuel Parra"})
```

You have new collections created:

```
show collections
```

## Delete collections

To remove a collection from the database:

```
db.MySecondCollection.drop();

```


## Working with documents on collections

To insert data into MongoDB collection, you need to use MongoDB's ``insert()`` or ``save()`` method.

Example (do not write): 

```
> db.MyFirstCollection.insert(<document>);
```

Example of document: a place.

```
{    
     "bounding_box":
    {
        "coordinates":
        [[
                [-77.119759,38.791645],
                [-76.909393,38.791645],
                [-76.909393,38.995548],
                [-77.119759,38.995548]
        ]],
        "type":"Polygon"
    },
     "country":"United States",
     "country_code":"US",
     "likes":2392842343,
     "full_name":"Washington, DC",
     "id":"01fbe706f872cb32",
     "name":"Washington",
     "place_type":"city",
     "url": "http://api.twitter.com/1/geo/id/01fbe706f872cb32.json"
}
```

To insert in ``MongoDB`` to ``MyFirstCollection``:

```
db.MyFirstCollection.insert(
{    
     "bounding_box":
      {
        "coordinates":
        [[
                [-77.119759,38.791645],
                [-76.909393,38.791645],
                [-76.909393,38.995548],
                [-77.119759,38.995548]
        ]],
        "type":"Polygon"
      },
     "country":"United States",
     "country_code":"US",
     "likes":2392842343,
     "full_name":"Washington, DC",
     "id":"01fbe706f872cb32",
     "name":"Washington",
     "place_type":"city",
     "url": "http://api.twitter.com/1/geo/id/01fbe706f872cb32.json"
}
);
```

Check if document is stored using ``.find()`` function:

```
> db.MyFirstCollection.find();
```


Now you can add multiple documents in a variable (like ``Javascript`` and ``json``):

```
	var places= [
		{    
	     "bounding_box":
	      {
	        "coordinates":
	        [[
	                [-77.119759,38.791645],
	                [-76.909393,38.791645],
	                [-76.909393,38.995548],
	                [-77.119759,38.995548]
	        ]],
	        "type":"Polygon"
	      },
	     "country":"United States",
	     "country_code":"US",
	     "likes":2392842343,
	     "full_name":"Washington, DC",
	     "id":"01fbe706f872cb32",
	     "name":"Washington",
	     "place_type":"city",
	     "url": "http://api.twitter.com/1/geo/id/01fbe706f872cb32.json"
	},
	{    
	     "bounding_box":
	      {
	        "coordinates":
	        [[
	                [-7.119759,33.791645],
	                [-7.909393,34.791645],
	                [-7.909393,32.995548],
	                [-7.119759,34.995548]
	        ]],
	        "type":"Polygon"
	      },
	     "country":"Spain",
	     "country_code":"US",
	     "likes":2334244,
	     "full_name":"Madrid",
	     "id":"01fbe706f872cb32",
	     "name":"Madrid",
	     "place_type":"city",
	     "url": "http://api.twitter.com/1/geo/id/01fbe706f87333e.json"
	}
	]
```

and, now insert the variable ``places``:

```
db.MyFirstCollection.insert(places)
```

In the inserted document, if we don't specify the ``_id`` parameter, then MongoDB assigns a unique ObjectId for this document.
You can override value `_id`, using your own ``_id``.

Two methods to save/insert:

```
db.MyFirstCollection.save({username:"myuser",password:"mypasswd"})
db.MyFirstCollection.insert({username:"myuser",password:"mypasswd"})
```

Differences:

>If a document does not exist with the specified ``_id`` value, the ``save()`` method performs an insert with the specified fields in the document.

>If a document exists with the specified ``_id` value, the ``save()`` method performs an update, replacing all field in the existing record with the fields from the document.


## Selecting/Querying/Filtering

Show all documents in ``MyFirstCollection``:

```
> db.MyFirstCollection.find();
```

Only one document, not all:

```
> db.MyFirstCollection.findOne();
```

Counting documents, add ``.count()`` to your sentences:

```
> db.MyFirstCollection.find().count();
```


Show documentos in pretty mode:

```
> db.MyFirstCollection.find().pretty()
```

Selecting or searching by embeded fields, for example ``bounding_box.type``:

```
...
 "bounding_box":
    {
        "coordinates":
        [[
                [-77.119759,38.791645],
                [-76.909393,38.791645],
                [-76.909393,38.995548],
                [-77.119759,38.995548]
        ]],
        "type":"Polygon"
    },
...
```


```
> db.MyFirstCollection.find("bounding_box.type":"Polygon")
```


Filtering:

Equality	``{<key>:<value>}``	 ``db.MyFirstCollection.find({"country":"Spain"}).pretty()``

Less Than	``{<key>:{$lt:<value>}}``	``db.mycol.find({"likes":{$lt:50}}).pretty()``

Less Than Equals	``{<key>:{$lte:<value>}}``	``db.mycol.find({"likes":{$lte:50}}).pretty()``

Greater Than	``{<key>:{$gt:<value>}}``	``db.mycol.find({"likes":{$gt:50}}).pretty()``

More: ``gte`` Greater than equal, ``ne`` Not equal, etc. 

AND:

```
> db.MyFirstCollection.find(
   {
      $and: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

OR:

```

> db.MyFirstCollection.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```


Mixing up :

```
db.MyFirstCollection.find(
		{"likes": {$gt:10}, 
		 $or: 
			[
			 {"by": "..."},
   			 {"title": "..."}
   			]
   		}).pretty()
```


Using regular expresions on fields, for instance to search documents where the name field
``name`` cointais ``Wash``.


```
db.MyFirstCollection.find({"name": /.*Wash.*/})

```


## Updating documents

Syntax:

```
> db.MyFirstCollection.update(<selection criteria>, <data to update>)
```

Example:

```
db.MyFirstCollection.update(
	 { 'place_type':'area'},
	 { $set: {'title':'New MongoDB Tutorial'}},
	 {multi:true}
	);
```

IMPORTANT: use ``multi:true`` to update all coincedences.


## Deleting documents

MongoDB's ``remove()`` method is used to remove a document from the collection. ``remove()`` method accepts two parameters. One is deletion criteria and second is justOne flag.

```
> db.MyFirstCollection.remove(<criteria>)
```

Example:

```
db.MyFirstCollection.remove({'country':'United States'})
```


## Import external data


Download this dataset your HOME (copy this link: https://raw.githubusercontent.com/DiCITS/MasterCiberSeguridad2018/master/MongoDB/convertcsv.csv):

[DataSet](https://raw.githubusercontent.com/DiCITS/MasterCiberSeguridad2018/master/MongoDB/convertcsv.csv) 7585 rows and 794 KB)



Use the next command (out of mongodb):

```
curl -O https://raw.githubusercontent.com/DiCITS/MasterCiberSeguridad2018/master/MongoDB/convertcsv.csv
```

or download from [github](https://raw.githubusercontent.com/DiCITS/MasterCiberSeguridad2018/master/MongoDB/convertcsv.csv).

To import this file (from shell) [Remember the path of the file you downloaded]:

```
mongoimport -d USERID -c <your collection> --type csv --file convertcsv.csv --headerline
```

Now, connect to mongo:

```
mongo
```

And use the your database:

```
use <YOURID>;
```

Check if collection is created and it have content:

```
show collections;
```

And list the content of the collection:

```
db.<your collection>.find();
```

It returns something like:

```
{ "_id" : ObjectId("5c3601e5144550c19ed16849"), "cdatetime;address;district;beat;grid;crimedescr;ucr_ncic_code;latitude;longitude" : "1/1/06 0:00;22 BECKFORD CT;6;6C        ;1443;476 PC PASS FICTICIOUS CHECK;2501;38.50677377;-121.4269508" }
{ "_id" : ObjectId("5c3601e5144550c19ed1684a"), "cdatetime;address;district;beat;grid;crimedescr;ucr_ncic_code;latitude;longitude" : "1/1/06 0:00;3421 AUBURN BLVD;2;2A        ;508;459 PC  BURGLARY-UNSPECIFIED;2299;38.6374478;-121.3846125" }
{ "_id" : ObjectId("5c3601e5144550c19ed1684b"), "cdatetime;address;district;beat;grid;crimedescr;ucr_ncic_code;latitude;longitude" : "1/1/06 0:00;4 PALEN CT;2;2A        ;212;10851(A)VC TAKE VEH W/O OWNER;2404;38.65784584;-121.4621009" }
{ "_id" : ObjectId("5c3601e5144550c19ed1684c"), "cdatetime;address;district;beat;grid;crimedescr;ucr_ncic_code;latitude;longitude" : "1/1/06 0:00;2217 16TH AVE;4;4A        ;957;459 PC  BURGLARY VEHICLE;2299;38.537173;-121.4875774" }
.......
```

### Praticing queries with MongoDB

Try out the next queries on your collection:

- Count number of incidents of the collection:

```
db.<your collection>.count();
```
- Count number of incidens where ``ucr_ncic_code`` is ``7000`` (``ASSAULT WITH WEAPON - I RPT``) of the collection:

```
db.<your collection>.find(....)
```



- Count number of crimes per hour.


## MongoDB Clients

- Command line tools: https://github.com/mongodb/mongo-tools
- Use Mongo from PHP: https://github.com/mongodb/mongo-php-library
- Use Mongo from NodeJS: https://mongodb.github.io/node-mongodb-native/
- Perl to MongoDB: https://docs.mongodb.com/ecosystem/drivers/perl/
- Full list of Mongo Clients (all languages): https://docs.mongodb.com/ecosystem/drivers/#drivers


# References 

- Getting Started with MongoDB (MongoDB Shell Edition): https://docs.mongodb.com/getting-started/shell/
- MongoDB Tutorial: https://www.tutorialspoint.com/mongodb/
- MongoDB Tutorial for Beginners: https://www.youtube.com/watch?v=W-WihPoEbR4
- Mongo Shell Quick Reference: https://docs.mongodb.com/v3.2/reference/mongo-shell/

