## Deploying WinkNodeRed and couchDB as docker containers on raspberry pi.

#### This documentation assumes that you already have docker installed. I recoommend to use Hypriot OS (https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/)
#### Also, this documentation shows how to use host local folders for persistence. You can mount a NFS/CIFS/ISCSI file system to your host and use that instead. There are plenty resources on how to mount NAS storage on the internet.

### Steps:
for the purpose of this documentation I assume that you use default hypriot OS user is <b>pirate</b> if you use raspbian jessie and your user id is <b>pi</b> or anything else - adjust steps below accordingly.
Also my assumtion is that you know how to copy files to unix box or how to create and edit files using either 'vi' or 'nano' editors


1. login to rasperry pi as user pirate
    * Optional but makes things easier: install portainer.io GUI for docker by running following command:<br>
```
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer --no-auth
```
    once completed, open your browser and navigate to http://your_rpi_ip_address:9000 . 
    you should see portainer.io UI
2. navigate to home directory cd /home/pirate
3. create a folder where you will keep settings.js and name it "settings"<br>
```
mkdir settings
```
4. create a folder where you will keep your node red flows and name it "flows"<br>
```
mkdir flows
```
5. create a folder where you will keep your couchDB database and name it "couchDB"<br>
```
mkdir couchDB
```
6. get settings.js from https://github.com/tfatykhov/local-wink-node-red/blob/master/docker/settings.js and copy settings.js (after you put your details) to /home/pirate/settings folder. 
	You can copy some stuff from your current settings.js but make sure you do not overwrite things as there are some changes.
7. execute following command to install couch DB<br>
```
docker run -d -p 5984:5984 -v /home/pirate/couchDB:/opt/couchdb/data -e COUCHDB_USER=wnr -e COUCHDB_PASSWORD=wnr --restart always --name wnr-couchdb rpidocker/rpi-couchdb
```
8. execute following command to install winknodered<br>
```
docker run -it -p 1880:1880 -v /home/pirate/settings:/settings -v /home/pirate/flows:/flows --restart always --name wnr tfatykhov/winknodered:rpi
```
* <b>Important</b>: if you already have node-red running on same pi, change the external port to 1990 (or any other one) for docker WNR container example below:
```
docker run -it -p 1990:1880 -v /home/pirate/settings:/settings -v /home/pirate/flows:/flows --restart always --name wnr tfatykhov/winknodered:rpi
```
you should see node red starting etc. you can then press <b>CtrlP</b> then <b>CtrlQ</b> to detach current terminal and let it run on background

9. once everyting is done, you can open http://your_docker_server_ip:1880/red and import your flows. 
	IMPORTANT!! - if you already have working WNR (locally or in bluemix) before deploying winkCore flow - disable "wink subscriptions" since your docker instance is not exposed outside by default
	in order to disable "subscription" prior to deploy double-click on inject node which connection goes to "prepare wink devices subscription request" function node change "repeat" to "none" and uncheck "inject once at start"
	then you can deploy everything.
10. In order to stop WNR you should execute: `docker stop wnr`
11. In order to start WNR you should execute: `docker start wnr`
12. To access your couchDB navigate to http://<docker_ip>:5984/_utils. user id/password to access - wnr/wnr
13. Every time you change settings.js - you need to stop WNR and start WNR (pp. 11 and 12)

### Adding https support

#### Assumtion is that you already have certificate generated (.key and .pem files)

1. Copy your .key and .pem files to /home/pirate/settings folder (the one you prepaired earlier)
2. modify settings.js, uncomment lines for https and replace values based on sample below repacing your_file_name with actual name of corresponding file:
```
   https: {
        key: fs.readFileSync('/settings/your_file_name.key'),
        cert: fs.readFileSync('/settings/your_file_name.pem')
   },
```
3. create port forwarding rule on your router using port number from item 8 and your docker host ip address
4. change settings.js and modify BlueMixUrlBase to https://your_external_domain:your_configured_port
5. <b>Important</b>. Since this instace also use coachDB, before restarting WNR, navigate to configuration tab of ui 1.0 and change BlueMix Url Base there as well or application will not function properly.
6. restart wnr docker container by running commands from pp 10 and  11 of previous section.
7. now open your node red using https://your_external_domain:your_port/red

