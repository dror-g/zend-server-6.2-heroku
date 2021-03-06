# Overview

This buildpack provides Zend Server 6.2 on Cloud Foundry. In the future, the Web server and Zend Server binaries will probably be moved into the stack. This will save space for each container, and make pushes and restarting almost instantaneous. To ensure portability and easy installation, they are currently included in the buildpack.

# Buildpack Components

* Zend Server 6.2 Enterprise edition.
* Zend Server 6.2 configuration files
* PHP 5.4
* Nginx web server
 

# Usage
1. Create a folder on your workstation, and "cd" into it.
2. Create an empty file called "zend_server_php_app". If you do not do this, you will have to manually specify which buildpack to use for the app. 
3. Make sure your app contains an "index.php" file.
4. Issue the `cf push --buildpack=https://github.com/davidl-zend/zend-server-mysql-buildpack-dev` command. Allocate at least 512M of RAM for your app. 
5. When prompted, save your manifest.
6. Optional - bind a MySQL service (cleardb/mysql/MariaDB/user-provided) to the app - this will cause Zend Server to operate in cluster mode (experimental). Operating in cluster mode enables: scaling, persistence of settings changed using the Zend Server UI and persistence of apps deployed using Zend Server's deployment mechanism. 
If you bind more than one database service to an app, specify which service Zend Server should use by setting the 'ZS_ DB' env variable to the correct service: `cf set-env <app_name> ZS_DB <db-service-name>`. Otherwise, Zend Server will use the first database available.
7. When prompted, save the manifest.
8. Issue the comand below to change the Zend Server UI password (this can be performed in the future in case you forget your password):
`cf set-env <app_name> ZS_ADMIN_PASSWORD <password>`
9. The previous steps should generate a YAML file named "manifest.yml" (see example below). Optional - add and push the generated manifest in future applications to facilitate smoother future pushes. 

 ```
 ---
 env:
    ZS_ACCEPT_EULA: 'TRUE'
    ZS_ADMIN_PASSWORD: '<password_for_Zend_Server_GUI_console>'
 applications:
 - name: <app_name>
    instances: 1
    memory: <at least 512M >
    host: <app_name>
    domain: <your_cloud_domain>
    path: .
 ```

10. Wait for the app to start.
11. Once the app starts, you can access the Zend Server UI at http://url-to-your-app/ZendServer (e.g. http://dave2.vcap.me/ZendServer) using username 'admin' and the password you defined in step 7. If you forgot to perform step 7, then the password is 'changeme'. 
12. If you chose to save the manifest in the previous steps, then you can issue the `cf push` command to udpate your application code in the future.

# Using an External Database Service
It is possible to bind an external database to the Zend Server app as a "user-provided" service. Doing so will enable persistence, session clustering, and more. 
To bind an external database:

1. Run `cf create-service`.
2. As a service type select "user-provided".
3. Enter a friendly name for the service.
4. Enter service paramaters. The required parameters are `hostname, port, password, name`, where 'name' is the database Zend Server will use for its internal functions.
5. Enter the paramaters of your external database provider in order.
6. Bind the service to your app `cf bind-service [service-name] [app-name]`.
7. The service will be auto-detected upon push. Zend Server will create the schema and enable clustering features.


# Known Issues
* Zend Server Code Tracing may not work properly in this version.
* Several issues might be encountered if you do not bind MySQL providing service to the app (cleardb/mysql/MaraiaDB):
 * You can change settings using the Zend Server UI and apply them - but they will not survive application pushes and restarts, nor will they be propagated to new application instances.
 * Application packages deployed using Zend Server's deployment mechanism (.zpk packages) will not be propagated to new app instances.
 * Zend Server will not operate in cluster mode.
* Application generated data is not persistent (this is a limitation of Cloud Foundry) unless saved to a third party storage provider (like S3). 
* MySQL is not used automatically - If you require MySQL then you will have to setup your own server and configure your app to use it.
* Each container has their own full copy of Zend Server at the moment so droplet size is about 161MB for an empty app (this should not be an issue in public cloud foundry platforms).
* If the application does not contain an 'index.php' file you will most likely encounter a "403 permission denied error".

# Local Installation
You can optionally install the cartridge in yout local Cloud Foundry environment if you want it to be available as a system buildpack. This enables cartridge "auto detection" and adds it to the menu of cartridges presented by the cf utility. If you do not wish to install, skip this stage and go directly to the usage section.

# Requirements
* Working Cloud Foundry v2 environment with dea_ng and gorouter
* The lucid64 Cloud Foundry stack should be installed and enabled - check settings in /vagrant/dea_ng/config/dea.yml
* xz compression utility - it is installed automatically by vagrant if you follow the guide below

The buildpack is tested against a system generated by the vagrant installer: http://blog.cloudfoundry.com/2013/06/27/installing-cloud-foundry-on-vagrant/

# Instructions
Clone the git repo into the buildpack directory on your dea_ng node using the command:
`git clone https://github.com/davidl-zend/zend-server-mysql-buildpack-dev`

Alternatively you can clone the repo into a Web server in your environment and specify the buildpack to the cf client. 
i.e.  `cf push --buildpack=http://url-to-cloned-repo` or   (you can save this value in the app's manifest.yml).

# Additional Resources
The following resources will help you understand Cloud Foundry concepts and workflows:
* For more info on getting started with Cloud Foundry: http://docs.cloudfoundry.com/docs/dotcom/getting-started.html
* How to add a service in Cloud Foundry: http://docs.cloudfoundry.com/docs/dotcom/adding-a-service.html
* How to design apps for the cloud: http://docs.cloudfoundry.com/docs/using/app-arch/index.html
* Cloud Foundry documentation: http://docs.cloudfoundry.com/
