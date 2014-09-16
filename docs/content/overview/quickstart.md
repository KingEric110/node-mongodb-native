---
aliases:
- /doc/installing/
date: 2013-07-01
menu:
  main:
    parent: getting started
next: /overview/introduction
prev: /overview/installing
title: QuickStart
weight: 20
---

QuickStart
==========
The quickstart guide will show you how to set up a simple application using node.js and MongoDB. It scope is only how to set up the driver and perform the simple crud operations. For more inn depth coverage we encourage reading the tutorials.

Create the package.json file
----------------------------
Let's create a directory where our application will live. In our case we will put this under our projects directory.

    mkdir myproject
    cd myproject

Create a **package.json** using your favorite text editor and fill it in.

    {
      "name": "myproject",
      "version": "1.0.0",
      "description": "My first project",
      "main": "index.js",
      "repository": {
        "type": "git",
        "url": "git://github.com/christkv/myfirstproject.git"
      },
      "dependencies": {
        "mongodb": "~2.0"
      },
      "author": "Christian Kvalheim",
      "license": "Apache 2.0",
      "bugs": {
        "url": "https://github.com/christkv/myfirstproject/issues"
      },
      "homepage": "https://github.com/christkv/myfirstproject"
    }

Save the file and return to the shell or command prompt and use **NPM** to install all the dependencies.

    npm install

You should see **NPM** download a lot of files. Once it's done you'll find all the downloaded packages under the **node_modules** directory.

Booting up a MongoDB Server
---------------------------
Let's boot up a MongoDB server instance. Download the right MongoDB version from [MongoDB](http://www.mongodb.org), open a new shell or command line and ensure the **mongod** command is in the shell or command line path. Now let's create a database directory (in our case under **/data**).

    mongod --dbpath=/data --port 27017

You should see the **mongod** process start up and print some status information.

Connecting to MongoDB
---------------------
Let's create a new **app.js** file that we will use to show the basic CRUD operations using the MongoDB driver.

First let's add code to connect to the server and the database **myproject**.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      db.close();
    });

Given that you booted up the **mongod** process earlier the application should connect successfully and print **Connected correctly to server** to the console.

Let's Add some code to show the different CRUD operations available.

Inserting a Document
--------------------
Let's create a function that will insert some documents for us.

    var insertDocuments = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.insert([
        {a : 1}, {a : 2}, {a : 3}
      ], function(err, result) {
        assert.equal(err, null);
        assert.equal(3, result.result.n);
        assert.equal(3, result.ops.length);
        console.log("Inserted 3 document into the document collection");
        callback(result);
      });
    }

The insert command will return a results object that contains several fields that might be useful.

* **result** Contains the result document from MongoDB
* **ops** Contains the documents inserted with added **_id** fields
* **connection** Contains the connection used to perform the insert

Let's add call the **insertDocuments** command to the **MongoClient.connect** method callback.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      insertDocuments(db, function() {
        db.close();
      });
    });

We can now run the update **app.js** file.

    node app.js

You should see the following output after running the **app.js** file.

    Connected correctly to server
    Inserted 3 document into the document collection

Updating a document
-------------------
Let's look at how to do a simple document update by adding a new field **b** to the document that has the field **a** set to **2**.

    var updateDocument = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.update({ a : 2 }
        , { $set: { b : 1 } }, function(err, result) {
        assert.equal(err, null);
        assert.equal(1, result.result.n);
        console.log("Updated the document with the field a equal to 2");
        callback(result);
      });  
    }

The method will update the first document where the field **a** is equal to **2** by adding a new field **b** to the document set to **1**. Let's update the callback function from **MongoClient.connect** to include the update method.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      insertDocuments(db, function() {
        updateDocument(db, function() {
          db.close();
        });
      });
    });

Remove a document
-----------------
Next lets remove the document where the field **a** equals to **3**.

    var removeDocument = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.remove({ a : 3 }, function(err, result) {
        assert.equal(err, null);
        assert.equal(1, result.result.n);
        console.log("Removed the document with the field a equal to 3");
        callback(result);
      });    
    }

This will remove the first document where the field **a** equals to **3**. Let's add the method to the **MongoClient.connect** callback function.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      insertDocuments(db, function() {
        updateDocument(db, function() {
          removeDocument(db, function() {
            db.close();
          });
        });
      });
    });

Finally let's retrieve all the documents using a simple find.

Find All Documents
------------------
We will finish up the Quickstart CRUD methods by performing a simple query that returns all the documents matching the query.

    var findDocuments = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.find({}).toArray(function(err, docs) {
        assert.equal(err, null);
        assert.equal(2, docs.length);
        console.log("Found the following records");
        console.dir(docs)
        callback(docs);
      });      
    }

This query will return all the documents in the **documents** collection. Since we removed a document the total documents returned is **2**. Finally let's add the findDocument method to the **MongoClient.connect** callback.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      insertDocuments(db, function() {
        updateDocument(db, function() {
          removeDocument(db, function() {
            findDocuments(db, function() {
              db.close();
            });
          });
        });
      });
    });

This concludes the QuickStart of connecting and performing some Basic operations using the MongoDB Node.js driver. For more detailed information you can look at the tutorials covering more specific topics of interest.