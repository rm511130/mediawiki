# MediaWiki
MediaWiki is a free software open source wiki package written in PHP, originally for use on Wikipedia. It is now also used by several other projects of the non-profit Wikimedia Foundation and by many other wikis, including the https://www.mediawiki.org/wiki/MediaWiki website, the home of MediaWiki.

# My Goal is to run MediaWiki on Cloud Foundry

More specifically, my goal is to cf push mediawiki version 1.27.4 to run on Pivotal Cloud Foundry. 
My target environment will be PWS (https://run.pivotal.io) because it's readily available and free to use during an initial trial period.

# My Sources

I started by following the instructions I found here:  https://github.com/pronoiac/mediawiki_on_pws
and the comments/corrections/suggestions found here: https://github.com/cloudfoundry/php-buildpack/issues/214
but you can simply follow the step-by-step section below.

# Step by Step Installation Process

You can simply follow these steps to achieve a successful installation of MediaWiki on PWS:

1) Git clone this repository on your local machine: ````$ git clone https://github.com/rm511130/mediawiki````

   Explanation: Step 1 is roughly equivalent to obtaining the https://releases.wikimedia.org/mediawiki/1.27/mediawiki-1.27.4.tar.gz 
   file and locally extracting it ````$ tar -xvzf mediawiki-1.27.4.tar.gz```` 
   However, I also followed the instructions found here https://github.com/cloudfoundry/php-buildpack/issues/214 which were 
   published by https://github.com/dmikusa-pivotal. These instructions involve the creation of additional directories and files
   which you will get by simply executing the ````$ git clone https://github.com/rm511130/mediawiki```` command.
   
   Make sure to ````$ cd mediawiki```` before proceeding to step 2. 

2) You need to edit and adapt the ````manifest.yml```` file to match your target PCF environment:

   In the example shown below, I'm creating a route that applies to PWS (e.g. using the cfapps.io domain) and I'm binding the mediawiki
   App to a MySQL Service with the instance name of ````mediawiki```` which we will create in step 3 before we ````cf push```` the code.

````    
---
applications:
- name: mediawiki
  routes:
  - route: media_wiki.cfapps.io
  memory: 512M
  buildpack: php_buildpack # https://github.com/cloudfoundry/php-buildpack
  #buildpack: https://github.com/cloudfoundry/php-buildpack#v4.3.35
  #buildpack: https://github.com/heroku/heroku-buildpack-php
  env:
    SECRET_KEY: changeme
    UPGRADE_KEY: changeme
  services:
  - mediawiki
````

3) Create the MySQL DB instance using the following command:  ````$ cf create-service cleardb spark mediawiki````
  
   Explanation: The ````cf create-service```` command creates an instance of MySQL and provides, using environment variables, 
   the jdbcUrl, uri, name, hostname, port, username and password necessary for the App to bind to it.
   
   For example, once your App is running you can try the command ````$ cf env mediawiki```` to see the following information:
     
     ````     
     "jdbcUrl": "jdbc:mysql://us-cdbr-iron-east-85.cleardb.net/ad_d43339bc3be8f67?user=ba86fcfffff102&password=644ffff",
     "uri": "mysql://ba86fcd9df3102:64426e58@us-cdbr-iron-east-85.cleardb.net:3306/ad_d47099bc3be8f67?reconnect=true",
     "name": "ad_d47099bc3be8f67",
     "hostname": "us-cdbr-iron-east-85.cleardb.net",
     "port": "3306",
     "username": "ba86fcfffff102",
     "password": "644ffff"
     ````
     
     Another possibility would be to connect to an external MySQL DB instance by using the ````$ cf cups```` command.
     
     For example:
     
     ````
     $ cf cups mediawiki -p '{"DB_URL":"jdbc:mysql://mylocal.db.net/ade8f67?user=ba86&password=6ffff"}'
     ````
  
  4) Now we're ready to ````$ cf push````
  
  ````
  $ cf push
Pushing from manifest to org Central / space development as rmeira@pivotal.io...
Using manifest file /work/mediawiki/manifest.yml
Getting app info...
Creating app with these attributes...
+ name:        mediawiki
  path:        /Users/rmeira/work/src/github.com/user/mediawiki
+ buildpack:   php_buildpack
+ memory:      512M
  services:
+   mediawiki
  env:
+   SECRET_KEY
+   UPGRADE_KEY
  routes:
+   media_wiki.cfapps.io

Creating app mediawiki...
Mapping routes...
Binding services...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 13.75 MiB / 13.75 MiB [==========================================================================] 100.00% 12s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloading php_buildpack...
   Downloaded php_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (29.4M)
   -------> Buildpack version 4.3.49
   WARNING: PHP version >=5.5.9 not available, using default version (5.6.33). In future versions of the buildpack, specifying a non-existent PHP version will cause staging to fail. See: http://docs.cloudfoundry.org/buildpacks/php/gsg-php-composer.html
   Installing Nginx
   Downloaded [file:///tmp/buildpacks/be700b0834751083210c8afe7e38e643/dependencies/https___buildpacks.cloudfoundry.org_dependencies_nginx_nginx-1.13.8-linux-x64-9585c5f4.tgz] to [/tmp]
   NGINX 1.13.8
   Installing PHP
   PHP 5.6.33
   Downloaded [file:///tmp/buildpacks/be700b0834751083210c8afe7e38e643/dependencies/https___buildpacks.cloudfoundry.org_dependencies_php_php-5.6.33-linux-x64-b7f9a39b.tgz] to [/tmp]
   Downloaded [file:///tmp/buildpacks/be700b0834751083210c8afe7e38e643/dependencies/https___buildpacks.cloudfoundry.org_dependencies_php_php-5.6.33-linux-x64-b7f9a39b.tgz] to [/tmp]
   Downloaded [file:///tmp/buildpacks/be700b0834751083210c8afe7e38e643/dependencies/https___buildpacks.cloudfoundry.org_dependencies_composer_composer-1.6.3-52cb7bbb.phar] to [/tmp]
   Class ComposerHookHandler is not autoloadable, can not call pre-install-cmd script
   Loading composer repositories with package information
   Installing dependencies from lock file
   Package operations: 20 installs, 0 updates, 0 removals
     - Installing wikimedia/composer-merge-plugin (v1.3.1): Downloading (100%)
     - Installing composer/semver (1.4.0): Downloading (100%)
     - Installing cssjanus/cssjanus (v1.1.2): Downloading (100%)
     - Installing liuggio/statsd-php-client (v1.0.18): Downloading (100%)
     - Installing mediawiki/at-ease (v1.1.0): Downloading (100%)
     - Installing oojs/oojs-ui (v0.17.1): Downloading (100%)
     - Installing oyejorge/less.php (v1.7.0.10): Downloading (100%)
     - Installing wikimedia/assert (v0.2.2): Downloading (100%)
     - Installing wikimedia/base-convert (v1.0.1): Downloading (100%)
     - Installing wikimedia/cdb (1.3.0): Downloading (100%)
     - Installing wikimedia/cldr-plural-rule-parser (v1.0.0): Downloading (100%)
     - Installing wikimedia/html-formatter (1.0.1): Downloading (100%)
     - Installing wikimedia/ip-set (1.1.0): Downloading (100%)
     - Installing psr/log (1.0.0): Downloading (100%)
     - Installing wikimedia/php-session-serializer (v1.0.3): Downloading (100%)
     - Installing wikimedia/relpath (1.0.3): Downloading (100%)
     - Installing wikimedia/running-stat (v1.1.0): Downloading (100%)
     - Installing wikimedia/utfnormal (v1.0.3): Downloading (100%)
     - Installing wikimedia/wrappedstring (v2.0.0): Downloading (100%)
     - Installing zordius/lightncandy (v0.23): Downloading (100%)
   Generating optimized autoload files
   Class ComposerHookHandler is not autoloadable, can not call pre-install-cmd script
   Loading composer repositories with package information
   Installing dependencies from lock file
   Nothing to install or update
   Generating optimized autoload files
   Class ComposerVendorHtaccessCreator is not autoloadable, can not call post-install-cmd script
   Class ComposerVendorHtaccessCreator is not autoloadable, can not call post-install-cmd script
   Finished: [2018-02-14 04:45:55.179047]
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (591K)
   Uploaded droplet (80.1M)
   Uploading complete
   Stopping instance 93509c1a-00d0-420a-ae52-66131276e7f6
   Destroying container
   Successfully destroyed container

Waiting for app to start...

name:              mediawiki
requested state:   started
instances:         1/1
usage:             512M x 1 instances
routes:            media_wiki.cfapps.io
last uploaded:     Tue 13 Feb 23:44:47 EST 2018
stack:             cflinuxfs2
buildpack:         php_buildpack
start command:     $HOME/.bp/bin/start

     state     since                  cpu    memory         disk       details
#0   running   2018-02-14T04:46:32Z   0.0%   116K of 512M   8K of 1G  
````

Note that some errors, for example ````Class ComposerVendorHtaccessCreator is not autoloadable````, did show up in the logs. I have to address these, but you can still proceed to check whether the MediaWiki site is up and running on PCF.

5) Take your favorite browser and try to access the route you defined in the  ````manifest.yml```` file:



  
     
     
     
  
