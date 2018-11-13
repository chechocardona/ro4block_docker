# ro4block_docker
Dockerized version of ro4block

To run the next tutorial you need to have at least Docker Engine and Docker Compose installed on your system. An easy way to do this is following the steps to install pre-requisites on the Hyperledger Composer Tutorials's website [Installing Prerequisites](https://hyperledger.github.io/composer/latest/installing/installing-prereqs.html)

## Install CORS plugin

Initially, the authentication process in orcid is performed in the frontend directly until the rest server is modified to perform authentication . For this reason, is necessary to install a CORS plugin for web navigator, some options can be:

* [CORS everywhere](https://addons.mozilla.org/es/firefox/addon/cors-everywhere/) for firefox
* [Moesif Origin](https://chrome.google.com/webstore/detail/moesif-origin-cors-change/digfbfaphojjndkpccljibejjbppifbc) for chrome

## Install Hyperledger Fabric

Run the following commands to start a Hyperledger Fabric network to deploy our business networks to.
`````
mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
tar -xvf fabric-dev-servers.tar.gz
export FABRIC_VERSION=hlfv12
./downloadFabric.sh
./startFabric.sh
`````
In case you already have this scripts on your system and have used them to deploy a business network, please follow the next steps to start a clean Hyperledger Fabric network.
`````
cd ~/fabric-dev-servers
export FABRIC_VERSION=hlfv12
./stopFabric.sh
./teardownFabric.sh
./downloadFabric.sh
./startFabric.sh
`````

## Run Docker Containers

To download our docker volume with all the files necessary to deploy our business network, run the following commands.
1. First, we will download the docker volume and generate an interactive container from which we will configure our business network:
`````
docker run -it --net="host" chechocardona/ro4block:1.0 /bin/bash
`````
2. Next we will import our business network card into the wallet running on the interactive shell:
`````
cd fabric-dev-servers/certificates/
composer card import -f PeerAdmin@fabric-network.card
`````
3. Next we install the Hyperledger Composer runtime onto the Peer nodes from our Hyperledger Fabric network. 
`````
composer network install -c PeerAdmin@fabric-network -a ../bforos@0.0.1.bna
`````
4. Now we start the blockchain business network. We run this command with sudo. 
`````
sudo composer network start --networkName bforos --networkVersion 0.0.1 -A admin -S adminpw -c PeerAdmin@fabric-network
`````
5. Lastly we import the business network administrator card, and start the rest server API running over https.
`````
composer card import -f admin@bforos.card
export COMPOSER_TLS=true
composer-rest-server -c admin@bforos -n never
`````
Launch your browser and go to the URL given [https://localhost:3000/explorer](https://localhost:3000/explorer) for interacting with it. Rest server generates an endpoint for each participant, asset and transaction of the business network definition.

It is possible that your browser blocks the connection due to lack of a certificate for the https connection. If this is the case you have to add the exception to allow communication to the localhost at port 3000. Following it is shown how this is done in Mozilla Firefox.

If your browser blocks the connection a warning like the following will be shown in Firefox:

![Security Warning](pictures/NotSecureConnection.png?raw=true "Not Secure")

If you click 'Advanced' the options below will be shown:

![Add Exception](pictures/AddException.png?raw=true "Add Exception")

Then click Add Exception and the next pop up message will ask for confirmation:

![Confirm](pictures/Confirm.png?raw=true "Confirm")

## Running the front end
Now, to run the application open a new terminal window and run the following commands one by one:
`````
docker run -it --net="host" chechocardona/ro4block:1.0 /bin/bash
npm start
`````
Now open a new tab on your browser and go to [https://localhost:4200](https://localhost:4200)
As mentioned before, with the REST server it is likely that your browser blocks the connection. In that case follow the intructions to add the exception one more time.

Once the app is loaded, log-in with ORCID account, however since we are in the testing fase of the frontend application any user must be registered in the orcid sandbox. Create a user in the [sandbox](https://sandbox.orcid.org/). After the user is created you can access the frontend. 

Steps to use the application
* 1. Search in Github, Figshare or Slideshare Repositories
* 2. Allow the application access to the elements of the user's repository.
* 3. The application must display all of the users repositories.
* 4. Under the user name at the far top-right you can acces a view of the personal wallet with an initial endowment of 10 points.
* 5. Claim any of your repositories and update the wallet and the claim transaction will reflect a new balance on your wallet and emit an event related to the change in the wallet as a result of the claim transaction. 
* 6. You can access all of the claimed research object from the left tabs.

## Cleaning Up
After testing the bna desgined with Composer and deployed onto Fabric it is important to tidy up by stopping fabric and erasing  the docker containers and images that you don't want to keep. 

1. Stop each one of the processes running by pressing Ctrl+c on each one of the terminal windows opened. Namely, the one with te REST server, the one with the Composer Playground API, and the one with the Frontend.

2. Run
`````
exit
`````
on each one of them. This will stop the containers created. If you want to delete the containers and the image downloaded you will have to identify each one of them by ID running:
`````
docker ps -a
docker images -a
`````
For the containers and the images, respectively. To remove a container copy the ID and paste it after the command:
`````
docker rm [ID]
`````
The same way goes for an image:
`````
docker rmi [ID]
`````

3. To stop fabric navigate to the folder where you initially started the Hyperledger Fabric network.
`````
./stopFabric.sh
./teardownFabric.sh
`````
Clear the docker cointainers.

`````
./teardownAllDocker.sh
`````
Select option 1- Kill and remove only the containers. Then delete the images created, 
`````
docker rmi $(docker images dev-* -q)
`````
