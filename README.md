Welcome to RETS Manager

Introduction
-----------------------------------------------------------------------------------------------------------------------------
RETS Manager is a production ready framework for reading, storing and syncing real-estate data including images.

With the help of a real-estate agent, this framework was built based on the Data Distribution Facility (DDF®) data structure which was created by the Canadian Real Estate Association (CREA). Reading, deleting and modifying thousands of transactional data and images were tested.

This framework can read raw RETS data in XML format and store them in a PostgreSQL or SQLite database. It also provides a full syncing ability were the database will be updated automatically in terms of deleting and modifying records.
For Media, the framework provides the ability to store the images on AWS S3 or locally. Image syncing abilities is also provided.

Additionally, geocoding support is provided to store the lng/lat of the addresses.

The framework is Django ready. A web interface can be easily attached to this framework to have a fully functional website.
It can be also used by researchers to fetch real-estate data and images directly into a database.

For RETS requests. The framework a local modified copy of the RETS client under:
https://github.com/refindlyllc/rets

DDF documenation from CREA can be found under:
https://www.crea.ca/wp-content/uploads/2016/02/Data_Distribution_Facility_Data_Feed_Technical_Documentation.pdf

RETS server setup
-----------------------------------------------------------------------------------------------------------------------------
If you are using the Canadian DDF, this framework is ready for deployment.

The framework uses the sample username and password provided by CREA for testing.

To access the actual MLS data you will require permission from CREA or you will need to be a CREA member (Real-Estate Agent)

Just go to ddf_manager/settings.py and enter your credentials (username and password)
Make sure the login_url is pointing to 'http://data.crea.ca/Login.svc/Login'

For data from other RETS server. Some code modifications will be required.
The web_app/models.py file has to be modified according to the structure of your RETS server.
Also, the ddf_manager/db_mapping.py file has to be adjusted.
Other changes in the ddf_manager might be required.

Enabling S3
------------------------------------------------------------------------------------------------------------------------------
To enable S3 for images go to ddf_manager/aws_settings.py and enter your AWS info.
Also, make sure that S3 is enabled under ddf_manager/setting.py by setting 's3_reader = True'

Local Media Storage
------------------------------------------------------------------------------------------------------------------------------
For local media storage make sure s3 is disabled 's3_reader = False'
Set the Media URL under ddf_manager/setting.py 'MEDIA_DIR'

Operating Modes
-------------------------------------------------------------------------------------------------------------------------------
This framework can be used in two modes:

#1- Fully automated updates using a combination of Docker-Compose + Redis + Celery. This enables automatic periodic updates.

To use in the fully automated mode, install docker-compose then build and run the containers:
--docker-compose build
--docker-compose up

Make sure to enable DOCKER=True under RETS_Manager/settings.py. This will switch the database from SQLite to PostgreSQL

Goto ddf_manager/ddf.py modify the settings of the celery worker including the update frequency

Digital Ocean provides a docker ready instance which can be used
https://marketplace.digitalocean.com/apps/docker

#2- By local deployment as Django app (For manual selective data fetching).

Simply use the project files as a Django app after installing all the requirements (requirements.txt).
Make sure to apply migrations to create the database then run (manage.py runserver)
If you see this documentation on the home page it means you have successfully deployed the app.

To read the data from the RETS server go to <django_app_server>/test/

Logging
-------------------------------------------------------------------------------------------------------------------------------
Detailed logging for the updates will be recorded under ddf_client.log

Project Structure
-------------------------------------------------------------------------------------------------------------------------------

First: /ddf_manager
This app contains the following two folder and files:

1.    /rets_lib/: a modified version of the tiny RETS reader from https://github.com/refindlyllc/rets/tree/master/rets
2.    /ddf_client/*: Is the core ddf reader, it uses rets_lib to read the data from the RETS server, it has the following files:

    a-    ddf_streamer.py: responsible for reading the transactional data from the RETS server. This includes retrieving the master-list for the ids of all the listings, reading listings since a specific time_stamp, and retirive specific listing by id.
    b-    ddf_s3.py: responsible for downloading images from DDF and save them to S3
    c-    ddf_media.py: responsible for downloading images from DDF and save them locally.
    d-    ddf_client.py, defines streamer and s3 classes to complete the data fetching.

3.    Manager.py: Connects the Django models and the media storage to the ddf_client. Manages all data and images syncing.

4.    db_write.py: Is a file used by manager.py to interface with Django Model.

5.    db_mapping.py: Is a file used by db_write to map the Python Dictionaries retrieved using the DDF Client to the appropriate fields in DB.

6.    db_summary.py: Used to add geolocation and other additional data.

7.    ddf.py: Used by Celery for running the DDF Read.

8.    ddf_logger.py: Logging settings.

9.    Settings.py: Contains DDF app settings.

Second: /web_app

This folder contains an initial Django app that can be expanded to a full web interface.

The app contains the models as per the CREA DDF structure under models.py.
