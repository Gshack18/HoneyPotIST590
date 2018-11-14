# HoneyPot IST_590
>HoneyPot assignment
In this assignment, you will stand up a basic honeypot and demonstrate its effectiveness at detecting and/or collecting data about an attack. Guided instructions for doing this using specific software are provided below, but you are free to take any approach you wish that demonstrates the following basic principles:

Successful configuration and deployment of a network-accessible honeypot server with two primary features:
An attack surface that is vulnerable or exposed in some way to network-based attacks
A network security feature such as an IDS configured to detect and log such attacks
Illustration of at least one attack against the honeypot that can be detected or logged in a way that captures information about the attack or the attacker

>Features of MHN

MHN is a Flask application that exposes an HTTP API that honeypots can use to:

-It also allows system administrators to:

-Download a deploy script

-Connect and register

-Download snort rules

-Send intrusion detection logs


>Which Honeypot(s) you deployed

Dionaea

Snort

>Any issues you encountered

The install of the MHN admin has issues installing so I had to look at [MHN Troubleshooting](https://github.com/threatstream/mhn/wiki/MHN-Troubleshooting-Guide) for help

Once I had Dionaea depoyed I wasn't seeing anything in the log files and left it over night and still saw nothing. Did some troubleshooting but decided to move on to another vm called "Snort".


>Steps I had to do to get everything up and running

To create the VM i had to use GCP command as shown to allow firewall access

```
$ gcloud beta compute firewall-rules create mhn-allow-admin --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:3000,tcp:10000 --source-ranges=0.0.0.0/0 --target-tags=mhn-admin
```
Next I had to create the vm itself with this command
```
$ gcloud compute instances create "mhn-admin" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-admin","http-server","https-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-admin"
```

```
$ sudo apt-get update
$ sudo apt-get install git -y
```
Now the issue part was this where I had to change the script from a dead repo to another one that worked

```
$ cd /opt
$ sudo git clone https://github.com/RedolentSun/mhn.git
$ cd mhn
$ sudo ./install.sh
```
Once done I had to configure the page as accordingly
```
===========================================================
MHN Configuration
===========================================================
Do you wish to run in Debug mode?: y/n n
Superuser email: YOUR_EMAIL@YOURSITE.COM
Superuser password: 
Server base url ["http://x.x.x.x"]: 
Honeymap url ["http://x.x.x.x:3000"]:
Mail server address ["localhost"]: 
Mail server port [25]: 
Use TLS for email?: y/n n
Use SSL for email?: y/n n
Mail server username [""]: 
Mail server password [""]: 
Mail default sender [""]: 
Path for log file ["mhn.log"]: 
```

From all this it was time to setup our honey pot with this command from the gcloud command
```
$ gcloud beta compute firewall-rules create mhn-allow-honeypot --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=all --source-ranges=0.0.0.0/0 --target-tags=mhn-honeypot
```
and now the VM
```
$ gcloud compute instances create "mhn-honeypot-1" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-1"
```
From all of this we were giving Ip's to connect to the admin console using our superuser we made before.
Now it was time to login to the console and setup a deploy to have it run sensors for the honeypots.
By doing this we had to click deploy at the top left and chooice which one we wanted to get the scripts for. I have tried to deploy Dionaea but the sensor wouldn't pick up anything once I pased the script from console into the ssh of the honeypot vm. Next I tried to use the Snort deploy into a new tiny vm instance and it worked flawlessly until it timed out and no solutions worked for me extending the timeout sessions for he admin server.

Once I had deployed my honey pot 1 running Dionaea I saw the sensor, but nothing was being picked up and I have checked if the right Dionaea was set and firewall rules were set. Nothing worked, so then I removed the VM and deploy Snort using the method above but renaming the VM to snort. The information was gathered and shown below.

To setup Snort I had to go into the google console and create a new VM and customize it to have ubuntu and I have named it "Snort" then saved it instead of using the gcloud command "gcloud compute instances create".

Once it was created I went into the MHN admin server via the ip i was given and went to deploy. Next I went the deploy dropdown to pick Snort and then copied the script into the new VM I just created named Snort via its ssh command line. 

Next I went back into the admin server and clicked sensors to view the snort. From this I started to see data then I clicked the attacks button on the top left and saw the attacks happening.

>A summary of the data collected: number of attacks, number of malware samples, etc.

![summary picture](https://github.com/Gshack18/HoneyPotIST590/blob/master/snort.jpeg)

>Any unresolved questions raised by the data collected
Why do people do this or are they just random bots on the internet probing everything.

>Additionally, include a json export of the data you collected in the repo, instructions for which can be found in the next section.

[json session file](https://github.com/Gshack18/HoneyPotIST590/blob/master/session.json)
