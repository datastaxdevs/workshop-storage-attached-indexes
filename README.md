## 🎓🔥 Scalable Indexing for Cassandra using DataStax Astra 🔥🎓

[![License Apache2](https://img.shields.io/hexpm/l/plug.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Discord](https://img.shields.io/discord/685554030159593522)](https://discord.com/widget?id=685554030159593522&theme=dark)

![Storage Attached Index Workshop](https://user-images.githubusercontent.com/23346205/96910093-f5ee7100-146c-11eb-9cfb-06ea3732c2bd.png)

Welcome to the 'Scalable Indexing for Cassandra using DataStax Astra' workshop! In this two-hour workshop, the Developer Advocate team of DataStax will explain the new Storage Attached Indexing (SAI) feature using Astra, the cloud based Cassandra-as-a-Service platform delivered by DataStax, to demonstrate how you can use them to add some much wanted flexibility to your Cassandra data model by querying outside of primary key fields.

**To date, SAI is currently supported on DataStax Astra and DataStax Enterprise 6.8.3+. There is a currently a [CEP](https://cwiki.apache.org/confluence/display/CASSANDRA/CEP-7%3A+Storage+Attached+Index) to bring this functionality into Open Source Apache Cassandra.**

It doesn't matter if you join our workshop live or you prefer to do at your own pace, we have you covered. In this repository, you'll find everything you need for this workshop:

- Materials used during presentations
- Hands-on exercises
- Workshop videos
  - [First workshop](https://youtu.be/????) [NAM Time]
  - [Second workshop](https://youtu.be/????) [IST Time]
- [Discord chat](https://bit.ly/cassandra-workshop)
- [Questions and Answers](https://community.datastax.com/)

## Table of Contents

| Title  | Description
|---|---|
| **Slide deck** | [Slide deck for the workshop](slides/Presentation.pdf) |
| **1. Create your Astra instance** | [Create your Astra instance](#1-create-your-astra-instance) |
| **2. Getting started with SAI** | [Getting started with SAI](#2-getting-started-with-sai-(storage-attached-index)) |



## 1. Create your Astra instance

`ASTRA` service is available at url [https://astra.datastax.com](https://dtsx.io/workshop). `ASTRA` is the simplest way to run Cassandra with zero operations at all - just push the button and get your cluster. `Astra` offers **5 Gb Tier Free Forever** and you **don't need a credit card** or anything to sign-up and use it. 

**✅ Step 1a. Register (if needed) and Sign In to Astra** : You can use your `Github`, `Google` accounts or register with an `email`.

Make sure to chose a password with minimum 8 characters, containing upper and lowercase letters, at least one number and special character

- [Registration Page](https://dtsx.io/workshop)

![Registration Image](https://github.com/datastaxdevs/shared-assets/blob/master/astra/login-1000.png?raw=true)

- [Authentication Page](https://dtsx.io/workshop)

![Login Image](https://github.com/datastaxdevs/shared-assets/blob/master/astra/signin-raw.png?raw=true)


**✅ Step 1b. Choose the free plan and select your region**

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/choose-a-plan-1000-annotated.png?raw=true)

- **Select the free tier**: 5GB storage, no obligation

- **Select the region**: This is the region where your database will reside physically (choose one close to you or your users). For people in EMEA please use `europe-west-1` idea here is to reduce latency.

**✅ Step 1c. Configure and create your database**

You will find below which values to enter for each field.

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/create-and-configure-annotated-1000.png?raw=true)

- **Fill in the database name** - `sa_index_db.` While Astra allows you to fill in these fields with values of your own choosing, please follow our reccomendations to make the rest of the exercises easier to follow. If you don't, you are on your own! :)

- **Fill in the keyspace name** - `sa_index`. It's really important that you use the name sa_index here in order for all the exercises to work well. We realize you want to be creative, but please just roll with this one today.

- **Fill in the Database User name** - `index_user`. Note the user name is case-sensitive. Please use the case we suggest here.

- **Fill in the password** - `index_password1`. Fill in both the password and the confirmation fields. Note that the password is also case-sensitive. Please use the case we suggest here.

- **Create the database**. Review all the fields to make sure they are as shown, and click the `Create Database` button.

You will see your new database `pending` in the Dashboard.

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/dashboard-pending-1000.png?raw=true)

The status will change to `Active` when the database is ready, this will only take 2-3 minutes. You will also receive an email address when it is ready.

**✅ Step 1d. View your Database and connect**

Let’s review the database you have configured. Select your new database in the lefthand column.

Now you can select to connect, to park the database, to access CQL console or Studio.

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/summary-1000.png?raw=true)


[🏠 Back to Table of Contents](#table-of-contents)

## 2. Getting started with SAI (Storage Attached Index)
**SAI** is short for **Storage Attached Indexes**, it allows us to build indexes on Cassandra tables that dramatically improve the flexibility of Cassandra queries.

For a **non-technical introduction** to **SAI**, have a look at this [recent blog post](https://www.datastax.com/blog/get-your-head-clouds-part-1-3-build-cloud-native-apps-datastax-astra-dbaas-now-aws-gcp).

To learn more about **SAI** from a **technical perspective**, have a look at our [docs on SAI](https://docs.datastax.com/en/storage-attached-index/6.8/sai/saiQuickStart.html). Honestly, these docs are pretty great IMO especially the [SAI FAQ](https://docs.datastax.com/en/storage-attached-index/6.8/sai/saiFaqs.html). Definitely take a moment to read through these to get a better understanding of how all of this works and even more examples on top of what we are presenting in this repo.

Now, let's get into some examples. The first thing we'll need is a table and some data to work with. For that we need to talk about my dentist, or really a contrived example of a client data model a dentist might need to use.

**✅ Step 2a. Navigate to the CQL Console and login to the database**

In the Summary screen for your database, select **_CQL Console_** from the top menu in the main window. This will take you to the CQL Console with a login prompt.

![astra cqlsh console](https://user-images.githubusercontent.com/23346205/97186856-35bc9d80-1778-11eb-80fd-df1a2f264a25.png)

Once you click the _`CQL Console`_ tab it will automatically log you in and present you with a `token@cqlsh>` prompt.


**✅ Step 2b. Describe keyspaces and USE `sa_index`**

Ok, you're logged in, and now we're ready to rock. Creating tables is quite easy, but before we create one we need to tell the database which keyspace we are working with.

First, let's **_DESCRIBE_** all of the keyspaces that are in the database. This will give us a list of the available keyspaces.

📘 **Command to execute**
```
desc KEYSPACES;
```
_"desc" is short for "describe", either is valid_

📗 **Expected output**

![desc keyspace output](https://user-images.githubusercontent.com/23346205/97188152-939db500-1779-11eb-8e47-0d6b4ebea74b.png)

Depending on your setup you might see a different set of keyspaces then in the image. The one we care about for now is **_sa_index_**. From here, execute the **_USE_** command with the **_sa_index_** keyspace to tell the database our context is within **_sa_index_**.

📘 **Command to execute**
```
use sa_index;
```

📗 **Expected output**

![use sa_index output](https://user-images.githubusercontent.com/23346205/97188451-e4151280-1779-11eb-98a2-f7292621f6ac.png)

Notice how the prompt displays `token@cqlsh:sa_index>` informing us we are **using** the **_sa_index_** keyspace. Now we are ready to create our tables.

**✅ Step 2c. Create a _`clients`_ table and insert some data**

Create the table.

📘 **Command to execute**

```SQL
CREATE TABLE IF NOT EXISTS clients (
    uniqueid uuid primary key,
    firstname text,
    lastname text,
    birthday date,
    nextappt timestamp,
    newpatient boolean,
    photo text
);
```

Insert some data into the table.

_We don't have real image URLs, so we're just using a placeholder string._

📘 **Commands to execute**
```SQL
INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (D85745B1-4BEC-43D7-8B77-DD164CB9D1B8, 'Alice', 'Apple', '1984-01-24', '2020-10-20 12:00:00', true, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (2A4F139F-0BBF-4A6F-B982-5400F11D2F2B, 'Zeke', 'Apple', '1961-12-30', '2020-10-20 12:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (DF649261-89CB-446B-9998-FFA2D17506F9, 'Lorenzo', 'Banana', '1963-09-03', '2020-10-20 13:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (808E6BBF-A0F4-4E4C-9C97-E36751D51A8B, 'Miley', 'Banana', '1969-02-06', '2020-10-20 13:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (3D458A4D-2F54-4271-BEDC-1FC316B3CC96, 'Cheryl', 'Banana', '1970-07-11', '2020-10-20 14:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (287AB6B4-1AA6-45DF-B6F8-2BE253B9AACE, 'Red', 'Currant', '1974-02-18', '2020-10-20 15:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (AB49D151-CC04-40DC-AEEA-0A4E5F59D69A, 'Matthew', 'Durian', '1976-11-11', '2020-10-19 12:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (783CE790-16B4-4645-B27C-4FDF3994A755, 'Vanessa', 'Elderberry', '1977-12-03', '2020-10-20 15:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (D23997E4-CCCB-46BB-B92F-0D4582A68809, 'Elaine', 'Elderberry', '1979-11-16', '2020-10-20 10:00:00', true, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (36C386C1-3C3B-49FC-81B1-391D5537453D, 'Phoebe', 'Fig', '1986-01-27', '2020-10-21 11:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (00FEE7EE-8F93-4C2E-A8BE-3ADD81235822, 'Patricia', 'Grape', '1986-06-24', '2020-10-21 12:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (B9DB7E99-AD1C-49B1-97C6-87154663AEF4, 'Herb', 'Huckleberry', '1990-07-09', '2020-10-21 13:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo) 
VALUES (F4DB7673-CA4E-4382-BDCD-2C1704363590, 'John-Henry', 'Huckleberry', '1979-11-16', '2020-10-21 14:00:00', false, 'imageurl');
```

**✅ Step 2d. Create a _`clients`_ table and insert some data**

Now let's take a look at the data we just inserted.

📘 **Command to execute**
```
SELECT * FROM clients;
```

📗 **Expected output**

![select from client table](https://user-images.githubusercontent.com/23346205/97189410-f2176300-177a-11eb-90ca-3604f113520f.png)

**✅ Step 2e. Create some indexes**

Ok great, we have data in our table, but remember we used **_`uniqueid`_** as our **primary key** when we created the table. If we want to query a single patient, we'd have to do that by the **_`uniqueid`_** column because that's our **partition key** _(don't forget, a single value in the primary key is always the partition key)_. 

But how would we search for users outside of using their unique ID's? We need to look for clients based on information they give us when they walk in the office. Namely, information like first and last name, or birthdate. Maybe a combination of those. Let's set up some indexes to do that.

_Don't worry about options in the below statements just yet. We'll get to that. For now, just execute the commands to create your indexes._

📘 **Commands to execute**
```SQL
CREATE CUSTOM INDEX IF NOT EXISTS ON clients(firstname) USING 'StorageAttachedIndex' 
WITH OPTIONS = {'case_sensitive': false, 'normalize': true };

CREATE CUSTOM INDEX IF NOT EXISTS ON clients(lastname) USING 'StorageAttachedIndex' 
WITH OPTIONS = {'case_sensitive': false, 'normalize': true };

CREATE CUSTOM INDEX IF NOT EXISTS ON clients(birthday) USING 'StorageAttachedIndex';
```

