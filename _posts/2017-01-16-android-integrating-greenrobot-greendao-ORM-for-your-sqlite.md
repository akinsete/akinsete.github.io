---
layout: post
title:  "Integrating Greendao into your Android Application"
date:   2017-01-16 07:07:19
categories: [android, sqlite, database]
comments: true
---

greenDAO is an open source Android ORM making development for SQLite databases fun again. It relieves developers from dealing with low-level database requirements while saving development time 
An ORM is a layer between the relational SQLite database and the object-oriented app code. The ORM allows the developer to use the database without the need to transform objects into a format suited for the relational database.

At the end of this tutorial you should be able to integrate GreenDao into your android application in less than 10 minutes.
<!--more-->

### Features of greenDao
* Maximum performance (probably the fastest ORM for Android); our benchmarks are open sourced too
* Easy to use powerful APIs covering relations and joins
* Minimal memory consumption
* Small library size (<100KB) to keep your build times low and to avoid the 65k method limit
* Database encryption: greenDAO supports SQLCipher to keep your user’s data safe
* Strong community: More than 5.000 GitHub stars show there is a strong and active community



# Step 1
If you have an app already you can skip this step otherwise Create an android project from Android Studio.
File->New->New Project

## Step 2
Go to your build.gradle(Module:app) app level gradle file add <b>'org.greenrobot:greendao:3.2.0'</b> to your dependencies then sync the gradle file

![Add greenDao to your dependencies ](/img/greendaoapp/greendaoappgradle.png)

### Step 3
Go to your build.gradle(Prioject:greendaoapp) project level gradle file and add <b>org.greenrobot:greendao-gradle-plugin:3.2.0</b> as a class path to your dependencies then sync the gradle file

![Add greenDao classpath to project level gradle file ](/img/greendaoapp/greendaoappprojectgradle.png)


### Step 4
In this step we are going to create a greenDao generator, This generator will be responsible for auto creating entities and dao files. If you 
want to read more about this visit [GreenDao Generator](http://greenrobot.org/greendao/documentation/generator/) 

* Create a module by going to File->New->Module
* Select "Java Library" and Click Next
* Give the module a name "Here we will use - <b>greenDaoGenerator</b>"
* Package name "Here we use - <b>com.greendao</b>"
* Java Class Name  "Here we use - <b>MyGenerator</b>"
* Click Finish
 
![adding generator module](/img/greendaoapp/greendao-app-creating-module.png)

You will notice your project folder structure now contains two modules. One <b>app</b> and the other <b>greendaogenerator</b>

![new-folder-structure](/img/greendaoapp/greendao-folder-structure.png)


### Step 5
Open the  gradle for the new module and add <b>org.greenrobot:greendao-generator:3.2.0</b> to the dependencies then sync.

![adding dependency to new generator module](/img/greendaoapp/greendao-app-new-module-add-dependencies.png)

### Step 6
Now we are going to modify our generator class so we can generate the dao files and entities (Tables). In this example we will be creating a table called <b>users</b>
with columns user_id,last_name,first_name,email.</br>
See updated generator class below.


