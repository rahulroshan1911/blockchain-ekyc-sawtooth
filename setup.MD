# Setup Blockchain-eKYC

[1. Getting started](#1-getting-started)

[2. Install Java and Maven](#2-install-java-and-maven)

[3. MongoDB Installation](#3-mongodb-installation)

[4. NodeJS Setup](#4-nodejs-setup)

[5. Python setup](#5-python-setup)

[6. Sawtooth Ubuntu Setup](#6-sawtooth-ubuntu-setup)

[7. Create sawtooth-java-tp.service](#7-create-sawtooth-java-tpservice)

[8. Sawtooth for 1 or 2 nodes](#8-sawtooth-for-1-or-2-nodes)

[9. Sawtooth for 3 or more nodes with PoET registration](#9-sawtooth-for-3-or-more-nodes-with-poet-registration)

[10. eKYC Application - single node setup](#10-ekyc-application---single-node-setup)

[11. eKYC Application - multi node setup](#11-ekyc-application---multi-node-setup)

[12. eKYC Admin Application](#12-ekyc-admin-application)

## 1. Getting started
You will need an Ubuntu 16.04 server with the following ports open - `22, 80, 61172, 27017, 15590, 3000, 8008, 4004, 8800, 2309 (MEMBER PORT), 2405 (ADMIN PORT)`.

Login to your machine `ssh <VM-USERNAME>@<VM-IP-ADDRESS>` and run following git clone command at ${HOME} path:
```		
sudo git clone https://github.com/Primechain/blockchain-ekyc-sawtooth.git
```
Note: You can check ${HOME} path by running: `echo ${HOME}`

## 2. Install Java and Maven
	
Run `java_installation.sh` file by running the following command at the available location (sawtooth-ekyc-member/shell-scripts);
```	
bash java_installation.sh
```
After running above command, to verify Java version installed run following command:
```
java -version
```
Output can be like this:
```
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```
Run following command to verify maven installation:
```	
mvn -version
```
Output can be like this:
```
Apache Maven 3.0.4
```	 

## 3. MongoDB Installation
	
Run `mongo_installation.sh` file by running following command at the available location (sawtooth-ekyc-member/shell-scripts)
```	
bash mongo_installation.sh
```	
To verify mongo version installed run following command
```		
mongod --version	
```	
To start MongoDB service
```	
sudo service mongod start
```			
To stop MongoDB service	
```	
sudo service mongod stop
```	

## 4. NodeJS Setup

Run `node_installation.sh` file by running following command at the available location (sawtooth-ekyc-member/shell-scripts)
```		
bash node_installation.sh
```	   
Check nodejs version by running command:
```		   
nodejs --version
```	

## 5. Python setup

Ubuntu 16.04 comes with both Python 2.7 and Python 3.5 by default. Check python version using following commands:
```		
python2 -V
python3 -V
```			
Then run `python_setup.sh` file by running following command at the available location (sawtooth-ekyc-member/shell-scripts)
```		
bash python_setup.sh
```		
	
## 6. Sawtooth Ubuntu Setup:
	
Run `sawtooth_ubuntu_setup.sh` file by running following command at the available location (sawtooth-ekyc-member/shell-scripts)
```		
bash sawtooth_ubuntu_setup.sh
```	
At any time after installation, you can view the installed sawtooth packages with the following command:
```	
dpkg -l '*sawtooth*'
```	  
To create the genesis block, run the following commands:
```	
sawtooth keygen
sudo sawadm keygen
sawset genesis
sudo -u sawtooth sawadm genesis config-genesis.batch
```			
To run Sawtooth components as service, run following commands in sequence:
```		
sudo systemctl start sawtooth-validator.service
sudo systemctl start sawtooth-rest-api.service
sudo systemctl start sawtooth-settings-tp.service
sudo systemctl start sawtooth-poet-validator-registry-tp.service 
```	
Note: Start this service only in case of PoET registration i.e., in case of 3 or more than 3 node network. After doing the configurations as given in step 9.
```			 
sudo systemctl start sawtooth-java-tp.service
```	
**NOTE: Refer 7 to create this service.**
		
To see status of the component run:
```				
sudo systemctl status <service-name>
```			
To stop the component run:
```			
sudo systemctl stop <service-name>
```			
			
## 7. Create 'sawtooth-java-tp.service':

Please check the example `sawtooth-java-tp.service` in /sawtooth-ekyc-member/javaprocessor/
	 
In file;
```	
[Service]
WorkingDirectory = /home/pcuser/blockchain-ekyc-sawtooth/sawtooth-ekyc-member/javaprocessor
ExecStart=/home/pcuser/blockchain-ekyc-sawtooth/sawtooth-ekyc-member/javaprocessor/build_java_processor.sh
```	
edit path accordingly to the path of `javaprocessor` and `build_java_processor.sh` as per your system.
		
Save and move the file to `/etc/systemd/system/`
	 
After this run the service by moving out of root user.

```
sudo systemctl start sawtooth-java-tp.service
```
	
	
## 8. Sawtooth for 1 or 2 nodes
 	
1. On the network, for the "FIRST NODE" only, it is necessary to create a genesis block. To do that, run following commands:
 
```	
sawtooth keygen	
sudo sawadm keygen
sawset genesis
sudo -u sawtooth sawadm genesis config-genesis.batch
```		
After the last command, the following output appears:
 
 ```		
 Processing config-genesis.batch...
 Generating /var/lib/sawtooth/genesis.batch
 ```
		
2. On other node, create user key and root key for the validator using following commands

```	
 sawtooth keygen
 sudo sawadm keygen	
```		
3. The validator configuration file specifies network information that allows the validator to advertise itself properly and search for peers. This file also contains settings for optional authorization roles and transactor permissions.
 If the config directory contains a file named validator.toml, the configuration settings are applied when the validator starts. By default, config directory is `/etc/sawtooth`.
	   
Refer to [this documentation](https://sawtooth.hyperledger.org/docs/core/releases/latest/sysadmin_guide/configuring_sawtooth/validator_configuration_file.html) to know configurations in `validator.toml` file. Find the attached `validator.toml.example` file at `/etc/sawtooth/` directory. After changing configurations to apply those changes, change file name to `validator.toml`]
	   
 In the example `validator.toml` file in /sawtooth directory:
```	   
bind = [
  	"network:tcp://10.1.0.6:8800",
  	"component:tcp://127.0.0.1:4004"
	]
```	   
`network` is the IP address of the node on which the configuations are done. [You can check IP address by running `ifconfig` command]
	   
  Similarly, configurations given in `seeds` and `peers` are the IP addresses of the other nodes in the network. Do the same 	configurations on all nodes. As given below, in case of 2 nodes there will be one peer/seed
```	   
seeds = ["tcp://10.1.0.7:8800"]	   
peers = ["tcp://10.1.0.7:8800"]
```	   
  Similarly, in case of 4 nodes there will be 3 peers/seeds for each node. In that case, configurations will be like 		     following:
```	   
seeds = ["tcp://10.1.0.7:8800","tcp://10.1.0.8:8800","tcp://10.1.0.9:8800"]	   
peers = ["tcp://10.1.0.7:8800","tcp://10.1.0.8:8800","tcp://10.1.0.9:8800"]
```	    
	   
4. After completing configurations successfully on both nodes, as shown in step 3; run the services on both nodes again.

5. To check validator logs, find `validator-debug.log` file in `/var/log/sawtooth` directory.
	
***Note:***
* Please find `validator.toml` example file in sawtooth directory. 
* In case of one node only,just do step 2.
	
## 9. Sawtooth for 3 or more nodes with PoET registration
 	
In case of PoET registration, follow below steps on first node (where genesis block is created):
```	
sawtooth keygen
sudo sawadm keygen
```		
Ensure that the required user and validator keys exist:
```
ls ~/.sawtooth/keys/
yourname.priv    yourname.pub
```
```
ls /etc/sawtooth/keys/
validator.priv   validator.pub
```

1. Create a batch to initialize the Settings transaction family in the genesis block.
```	
sawset genesis -k /etc/sawtooth/keys/validator.priv -o config-genesis.batch
```
2. Create a batch to initialize the PoET consensus settings. This command sets the consensus algorithm to PoET simulator, and 	      	 then applies the required settings.
```	
sawset proposal create -k /etc/sawtooth/keys/validator.priv \
-o config.batch \
sawtooth.consensus.algorithm=poet \
sawtooth.poet.report_public_key_pem="$(cat /etc/sawtooth/simulator_rk_pub.pem)" \
sawtooth.poet.valid_enclave_measurements=$(poet enclave measurement) \
sawtooth.poet.valid_enclave_basenames=$(poet enclave basename)
```
3. Create a batch to register the first validator with the PoET Validator Registry. Without this command, the validator would 	              	 not be able to publish any blocks.
```	
poet registration create -k /etc/sawtooth/keys/validator.priv -o poet.batch
```		
4. Create a batch to configure optional PoET settings. This example shows the default settings.
```	
sawset proposal create -k /etc/sawtooth/keys/validator.priv \
-o poet-settings.batch \
sawtooth.poet.target_wait_time=30 \
sawtooth.poet.initial_wait_time=300 \
sawtooth.publisher.max_batches_per_block=500 \
sawtooth.poet.ztest_minimum_win_count=10000 \
sawtooth.poet.key_block_claim_limit=100000
```	
5. Combine the previously created batches into a single genesis batch that will be committed in the genesis block.
```	
sawadm genesis config-genesis.batch config.batch poet.batch poet-settings.batch
```		
6. Then run all the services given in step 6 in sequence including `sawtooth-poet-validator-registry-tp.service` on all nodes. Please refer to [this documentation](https://sawtooth.hyperledger.org/docs/core/nightly/master/app_developers_guide/creating_sawtooth_network.html) for more details.
 	
## 10. eKYC Application - single node setup

***Please make sure all the services are up and running before starting on following steps.***
```		
cd /sawtooth-ekyc-member/jsclient/configs	
sudo nano notification_config.js
```	
Provide your ip address at IP_ADDRESS: "VM-IP-ADDRESS" eg., IP_ADDRESS : "198.168.0.1"
Similarly, provide other configurations provided in the `notification_config.js` file. 
	   
Please enter SENDGRID_API_KEY in config.conf. This is your Sendgrid API key. This is for transactional emails. If you don't have a sendgrid account, create one at https://sendgrid.com/
	   
Save and exit nano.
		
Now run:

```
bash category.sh
```		
In /sawtooth-ekyc-member directory path;
```	
cd /jsclient
rm -rf node_modules
npm install --save
```		
Now run the application by running command [Check if Mongo Service is running or not. If not, start the service as  mentioned in MongoDB installation]
```   
node kycMember.js
```   		

Go to `http://<VM-IP-ADDRESS>:2309/register` and fill up the admin account details:
```
Name
Email Address
Designation
Organisation
CIN
Website Address
Cell Number 
```
Click `create`. You will be redirected to a page that displays the message `Account activation mail has been sent to email`.
   
Click on `Activate Account` button in the email received for setting up this solution. You will be redirected to a page that displays the message - `Login credentials has been sent to your email`.
   
Go to your email and click on the link and use the login credentials emailed to login.
      
## 11. eKYC Application - multi node setup

***Please make sure all the services are up and running before starting on following steps.***
```	
cd /sawtooth-ekyc-member/jsclient/configs
sudo nano notification_config.js
```	
Provide your ip address at IP_ADDRESS: "VM-IP-ADDRESS" eg., IP_ADDRESS : "198.168.0.1".
Similarly, provide other configurations provided in the notification_config.js file. 
	   
Please enter SENDGRID_API_KEY in config.conf. This is your Sendgrid API key. This is for transactional emails. If you don't have a sendgrid account, create one at https://sendgrid.com/
	   
```Save and exit nano.```
		
Now run: [***Note: Run only on one of the nodes***] (sawtooth/).
```		
bash category.sh
```		
In /sawtooth-ekyc-member directory path;
```
cd /jsclient
rm -rf node_modules
npm install --save
```

This application is on shared database. So, before running application on multi-node network make sure it is pointing to    	the same mongo database. As given above, please change the mongo configurations in 'notification_config.js'.
		
Now run the application on all nodes by running command [Check if Mongo Service is running or not.If not, 
start the service as mentioned in MongoDB installation];

```
node kycMember.js
```		
Go to `http://<VM-IP-ADDRESS>:2309/register` and fill up the admin account details:

Name
Email Address
Designation
Organisation
CIN
Website Address
Cell Number

Click `create`
		
You will be redirected to a page that displays the message `Account activation mail has been sent to email`.
   
Click on `activate account` button in the email received for setting up this solution. You will be redirected to a page 	that displays the message - `Login credentials has been sent to your email`.
   
Go to your email and click on the link and use the login credentials emailed to login.

## 12. eKYC Admin Application 

[Even in case of multi-node network, run admin application on one node only.]:

Admin needs to approve members. Just start the application by running following commands:

```
cd sawtooth-ekyc-admin
```	
**NOTE: Please make sure it is pointing to the same mongo service as above** 

```	
node kycAdmin.js
```	
Go to `http://<VM-IP-ADDRESS>:2405/register` and fill up the admin account details:

Name
Email Address
Designation
Organisation
Cell Number

Click `create`
		
You will be redirected to a page that displays the message `Account activation mail has been sent to email`. 
	
Click on `activate account` button in the email received for setting up this solution. You will be redirected to a page that displays the message - `Login credentials has been sent to your email`.
	
Go to your email and click on the link and use the login credentials emailed to login.
