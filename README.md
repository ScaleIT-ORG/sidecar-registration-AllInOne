# sidecar-registration
Sidecar Application Registration on ETCD Key/Value Store


This repo shows how to add a simple sidecar application to your Main-application to register at ETCD (Key/Value Store https://coreos.com/etcd/ ) with Name, Icon and a little Description

This Repo also contains the ETCD-Browser container where you can visualize your etcd store at http://localhost:8080

## How to Use:
1. Start Containers:

		- docker-compose build
		- docker-compose up

2. Check ETCD/Application

		- Visit http://localhost:8080
		(You should see an entry called "Example1" with Name, Icon and Desc Values)
		- Visit http://localhost:3000
		(You should see a simple webpage from the example application)

3. Verify (Optional)
	
		Kill the Example Application and Check ETCD again to verify that the Example Application is not registrated anymore
		- docker ps  (get container_ID of Example application)
		- docker stop "container_ID"
		- Visit http://localhost:8080

## Explanation

- docker-compose.yml 

		Contains ETCD, ETCD-Browser and Example Application docker containers
- etcd-browser/
		
		This directory contains the ETCD-Browser Container where you can visualize your etcd store

- example/
	
		This is your example application which get registered at the ETCD storage.

- example/Dockerfile

		Describes a simple Node:boron Docker Container which executes the sidecar script at startup

- example/example.js

		Simple nodejs Server which serves a example.html page in wwwroot/

- example/sidecar.sh

	This is where the Magic happens

	At first it checks if ETCD is up and Running or waits till it gets a healthy signal from the ETCD Storage 

	```bash
	#check if etcd is up and running
	STR='"health": "false"'
	STR=$(curl -sb -H "Accept: application/json" "http://etcd:2379/health")
	while [[ $STR != *'"health": "true"'* ]]
	do
	        echo "Waiting for etcd ..."
	        STR=$(curl -sb -H "Accept: application/json" "http://etcd:2379/health")
	        sleep 1
	done
	```

	Then the application registers its keys

	```bash
	#Register Application
	curl -L -X PUT http://etcd:2379/v2/keys/Example1/url -d value="localhost:3000"
	curl -L -X PUT http://etcd:2379/v2/keys/Example1/icon -d value="/icon/favicon.png"
	curl -L -X PUT http://etcd:2379/v2/keys/Example1/desc -d value="Description here  ...."
	```

	Additionaly there is a SIGTERM handler which catches the ctr+c command and unregisters the application with:

	```bash

	curl -L -X PUT 'http://etcd:2379/v2/keys/Example1?recursive=true' -XDELETE

	```

	The script also starts the main Application and executes a endless loop to catch the ctr+c Signal while the container is up and running

	```bash
	#run application
	node example.js &

	# wait forever
	while true
	do
	  tail -f /dev/null & wait ${!}
	done
	```
