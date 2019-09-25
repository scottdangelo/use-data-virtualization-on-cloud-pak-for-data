---
draft: true
code_series_toc: '/'

authors:
  - email: 'scott.dangelo@ibm.com'
    name: "Scott D'Angelo"

pta:
  - "cognitive, data, and analytics"

pwg:
  - "cloud pak for data"

components:
  - cloud-pak-for-data

completed_date: '2019-09-25'
last_updated: '2019-09-25'

abstract: "Use Data Virtualization on IBM Cloud Pak for Data to make queries across multiple data source quickly and easily."
exerpt: "Use Data Virtualization on IBM Cloud Pak for Data to make queries across multiple data source quickly and easily."
meta_description: "Use Data Virtualization on IBM Cloud Pak for Data to make queries across multiple data source quickly and easily."

meta_keywords: 'watson, learning path'
primary_tag: artificial-intelligence

tags:

meta_title: "Make queries with Data Virtualization"
title: "Make queries with Data Virtualization"
subtitle: ""
type: tutorial

## What you're going to learn

This tutorial is composed from the following steps:

1. [Get the data](#1-get-the-data)
1. [About the data set](#2-about-the-data-set)
1. [Seeding our Db2 Warehouse database](#3-seeding-our-db2-warehouse-database)
1. [Creating a new Cloud Pak for Data project](#4-creating-a-new-cloud-pak-for-data-project)
1. [Add a new Data Source connection](#5-add-a-new-data-source-connection)
1. [Assign virtualized data to your project](#6-assign-virtualized-data)

## 1. Get the data

The data that this tutorial uses to virtualize are located in a [github repository](https://github.ibm.com/Scott-DAngelo/data-virtualization-tutorial). To download you can clone the repository, use *wget* or download a *.zip* file.

Either run the following command:

```bash
git clone https://github.com/virtualizing-db2-warehouse-data-with-data-virtualization
cd virtualizing-db2-warehouse-data-with-data-virtualization
```
or run:

```bash
wget https://raw.githubusercontent.com/IBM/virtualizing-db2-warehouse-data-with-data-virtualization/billing.csv
wget https://raw.githubusercontent.com/IBM/virtualizing-db2-warehouse-data-with-data-virtualization/customer-service.csv
wget https://raw.githubusercontent.com/IBM/virtualizing-db2-warehouse-data-with-data-virtualization/products.csv
```

Alternatively, go to the [GitHub repo for this tutorial](https://github.com/IBM/virtualizing-db2-warehouse-data-with-data-virtualization) and download the archived version of the tutorial and extract it on your laptop.

![download tutorial zip](images/DownloadDataVisualization.png)

## 2. About the data set

The data set used for this tutorial is originally from Watson Analytics and was used on a [Kaggle](https://www.kaggle.com/blastchar/telco-customer-churn) project, it contains information about customer churn for a Telecommunications company. The data is split into three CSV files.

### **billing.csv**

This file has the following attributes:

* Customer ID
* Contract *(Month-to-month, one year, two year)*
* Paperless Billing *(Yes, No)*
* Payment Method *(Bank transfer, Credit card, Electronic check, Mailed check)*
* Monthly Charges *($)*
* Total Charges *($)*
* Churn *(Yes, No)*

### **customer-service.csv**

* Customer ID
* Gender *(Male, Female)*
* Senior Citizen *(1, 0)*
* Partner *(Yes, No)*
* Dependents *(Yes, No)*
* Tenure *(1-100)*

### **products.csv**

* Customer ID
* Phone Service *(Yes, No)*
* Multiple Lines *(Yes, No, No phone service)*
* Internet Service *(DSL, Fiber optic, No)*
* Online Security *(Yes, No, No internet service)*
* Online Backup *(Yes, No, No internet service)*
* Device Protection *(Yes, No, No internet service)*
* Tech Support *(Yes, No, No internet service)*
* Streaming TV *(Yes, No, No internet service)*
* Streaming Movies *(Yes, No, No internet service)*

## 3. Seeding our Db2 Warehouse database

We'll need a place to store our data. For this tutorial we've opted to use Db2 Warehouse on IBM Cloud for a few reasons: it simulates a realistic enterprise database, a free tier is provided by IBM Cloud, and we can easily load our data set.

### Log in and provision a Db2 Warehouse database

Log into (or sign up for) [IBM Cloud](https://cloud.ibm.com).

![IBM Cloud login](images/gcibm-cloud-sign-up.png)

From the dashboard click on the *Create resource* button to go to the *Catalog*.

![IBM Cloud Dashboard](images/gcibm-cloud-dashboard.png)

Find the [Db2 Warehouse](https://cloud.ibm.com/catalog/services/db2-warehouse) tile from the *Database* section.

![Db2 Warehouse in the Catalog](images/db2-0-catalog.png)

Choose the `Entry` plan as it is compatible with a `Lite` account.

> **NOTE**: There may be a message indicating the service will be charged, please disregard that message as there is no charge for staying under 1 GB of data.

![Entry level plan](images/db2-0-pricing.png)

Once the Db2 Warehouse service is created click on *Open Console*.

![A Db2 Warehouse instance has been provisioned](images/db2-1-cloud-launch.png)

You'll be directed to a Db2 web console dashboard where you can load data by clicking the *Load* button.

### Load the Db2 Warehouse database

![The Db2 Warehouse console](images/db2-2-console-overview.png)

Select the `billing.csv` file.

![Find data to load](images/db2-3-csv-find.png)

On the next panel we configure where the data will go in the database. We'll use the schema that was created for our database, it'll look like *DASHXXXX* where *XXXX* is a randomly generated number. Select that and click `+ New Table` and give it the name `Billing` to create a new table.

![Select a table name](images/db2-4-csv-config.png)

Accept the defaults on the next screen and click `Next`.

![Keep all defaults](images/db2-keep-defaults.png)

On the next panel click *Begin Load* to start loading the data.

![Begin loading the data](images/db2-5-csv-preload.png)

Verify all the rows were loaded. Click `Load More Data` to continue.

![Data has been loaded!](images/db2-6-csv-loaded.png)

Repeat the process for `products.csv` and `customer-service.csv`, call these tables `PRODUCTS` and `CUSTOMERS`. Note that there will be an additional panel to configure data, the defaults are accetable.

![Repeat the process for the other data sets](images/db2-8-csv-config-products.png)

### Jot down the credentials

Before we go to Cloud Pak for Data, we need to create credentials for our Db2 Warehouse by going back to our service, clicking on the *Service credentials* button, and creating a new credential. Copy these down somewhere as we'll need them in the next section.

![Db2 Warehouse credentials](images/db2-cloud-credentials.png)

## 4. Creating a new Cloud Pak for Data project

At this point of the tutorial we will be using Cloud Pak for Data for the remaining steps.

### Log into Cloud Pak for Data

Launch a browser and naviate to your Cloud Pak for Data deployment

![Cloud Pak for Data login](images/cpd-login.png)

### Create a new project

Go the (☰) menu and click *Projects*

![(☰) Menu -> Projects](images/cpd-projects-menu.png)

Click on *New project*

![Start a new project](images/cpd-new-project.png)

Create a new project, choose `Analytics project`, give it a unique name, and click *OK. Click `Create` on the next screen.

![Pick a name](images/cpd-new-project-name.png)

## 5. Add a new Data Source connection

For Cloud Pak for Data to read our Db2 Warehouse data we need to add a new *Data Source* to Cloud Pak for Data. This requires inputting the usual JDBC details.

To add a new data source, go to the (☰) menu and click on the *Connections* option.

![(☰) Menu -> Collections](images/cpd-conn-menu.png)

At the overview, click *Add connection*.

![Overview page](images/conn-1-overview-empty.png)

Start by giving your new *Connection* a name and select *Db2 Warehouse on Cloud* as your connection type. More fields should apper. Fill the new fields with the same credentials for your own Db2 Warehouse connection from the previous section (or ask your instructor for shared credentials). Click `Test Connection` and, after that succeeds, click `Add`.

![Add a Db2 Warehouse on Cloud connection](images/conn-2-details.png)

The new connection will be listed in the overview.

![Connection has been added!](images/conn-3-overview-db2.png)

## 2. Assign virtualized data

> **IMPORTANT**: A note to the instructors of this tutorial. At this point go to the [Admin Guide](images/gcadmin-guide/README.md#virtualize-db2-data-with-data-virtualization) and follow the `Virtualize Db2 data with Data Virtualization` section.

For this section we'll now use the Data Virtualization tool to import the data from Db2 Warehouse, which is now exposed as an Connection in Cloud Pak for Data.

### Assign the data to your project

From the menu click on *Collections -> Virtualized Data*, you'll be brought to the *My data* section. Here you should see the data that the administrator has assigned to you. Choose the three data sets available and click *Assign* to start importing it to your project.

![Select the data you want to import](images/dv-8-select-data.png)

From here, choose the project you previously created.

![Assign the data to a project](images/dv-9-assign.png)

Switching to our project should show all three virtualized tables, and two joined tables. Do not go to the next section until this step is performed.

![Our data sets at the end of this section](images/dv-project-data-all.png)

## Conclusion
