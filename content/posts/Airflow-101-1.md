---
title: "Airflow 建置測試-Part 1"
date: 2019-08-24T22:41:55+08:00
draft: false
description:
categories: 
 - airflow
featured_image:
author: "Leo"
---



## Airflow 建置測試-Part 1

![](https://cdn-images-1.medium.com/max/7680/1*5NM9hHD8J1FhCz84uy_8dw.jpeg)

*由於一直以來公司所用的ETL工具皆是像SSIS, Trinity 這樣類型的, 對於開源的Airflow 並沒有接觸過。在換了份新工作後，剛好有機會去試像Airflow 這類的 ETL工具。*

*在屢次安裝Airflow 失敗後，終於在Amazon Ubuntu 上安裝成功 Airflow 。為了不忘記這痛苦的過程 ，就藉此寫下較詳細的安裝步驟給同樣像我這樣的小白參考參考。* *(簡稱Airflow 踩坑記)*

 <iframe src="https://giphy.com/embed/2vlmXyWEEDm7korr7N" width="480" height="260" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/ok-k-you-got-it-2vlmXyWEEDm7korr7N">via GIPHY</a></p>

安裝環境 : Amazon EC2 — Ubuntu Server 18.04 LTS (HVM)

## Preparation List

* Python 3.6

* pip

* Mysql-server

* Mysql-client

* libmysqlclient-dev

* python-dev

* airflow-apache

<iframe src="https://giphy.com/embed/YLHwkqayc1j7a" width="480" height="288" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/empire-muslim-ottoman-YLHwkqayc1j7a">via GIPHY</a></p>

### Installing Database Backend (Mysql)

sudo apt-get install mysql-server

apt-get isntall mysql-client

sudo apt-get install libmysqlclient-dev

**Create airflow database**

On Airflow default setting is initdb, but official recommend using Mysql/Postgresql DB

mysql -u root

mysql> create database airflow **default** charset utf8 collate utf8_general_ci;

mysql> create user airflow@'localhost' identified by 'airflow';

mysql> grant all on airflow.* to airflow@'localhost';

mysql> flush privileges;

**Install python environment**

sudo apt install python3-pip

sudo pip3 install --upgrade pip

## Airflow Installation

pip install MySQL**-**python

### Set Up Airflow Default Home

mkdir -p ~/airflow

export AIRFLOW_HOME=~/airflow

export SLUGIFY_USES_TEXT_UNIDECODE=yes

### Installing Airflow

pip install "airflow[postgres, mysql, celery, rabbitmq, slack, crypto]"

### Check [here ](https://airflow.incubator.apache.org/installation.html#extra-packages)or more details on the list of the sub-packages and what they enable.

After successful installation, wee need to start Airflow’s database. Airflow requires a database to be initiated before you can run tasks.

The default db setting is SQLite, but in here we will change to use Mysql database. If in your airflow folder cannot find airflow.cfg file, you need to start airflow initdb first.

airflow initdb

The command will create airflow.cfg file under the airflow folder.

*Note: Make sure to have specified explicit_defaults_for_timestamp=1 in your my.cnf under [mysqld] or like this*

![](https://cdn-images-1.medium.com/max/2000/1*snJUWLXAk9L6-iu9StieWQ.png)

**Edit airglow.cfg file to change to Mysql database**

![](https://cdn-images-1.medium.com/max/2304/1*z58AH830PsgPm7eYIKXOag.png)

**Initiating Airflow Database**

airflow initdb

Now, your airflow database is changing to Mysql.

**Starting Airflow web server**

*Before start the webserver, make sure your Default Home is correct. Enter this script on the airflow folder.*

*export AIRFLOW_HOME=~/airflow*

airflow webserver -p 8080

**Starting Airflow Scheduler**

airflow scheduler

Now open the website: *localhost:8080* you will see the airflow web

![](https://cdn-images-1.medium.com/max/3652/1*k_TvmQO4wIaP1GfRN0ig9Q.png)

![](https://cdn-images-1.medium.com/max/3200/1*ptAfzDodd3GFPLS05da2xA.png)

Finally, we successful installed Airflow, and there are some function you might want to change in the airflow.cfg file. In the next topic, I will cover how to build Airflow Docker in 10 min and what else we have to change in airflow.cfg file.
