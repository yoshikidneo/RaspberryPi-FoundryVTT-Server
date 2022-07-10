# RaspberryPi-FoundryVTT-Server
![image](https://user-images.githubusercontent.com/70184841/153284283-aa073c52-326a-4871-9646-8ef87afc0aaf.png)

This is the step by step for my video, with links to other videos, hopefully this makes things easier especially for copy and pasting!

## Raspberry Pi Setup ##
First things first, setup your Raspberry Pi with any OS you wish, I was using Twister OS but have switched to the latest Raspbein 32bit since it is running on Linux 11, which is necessary for FoundryVTT v9 or later. 
Rasbian OS has officially launched the 64bit version which allows you to utilize more than 4gb RAM, commands should be pretty much the same although I haven’t tried it myself.
- [Raspberry Pi Install and Inital Setup](https://www.youtube.com/watch?v=BpJCAafw2qE)
- [Booting from SSD or Thumbdrive](https://www.youtube.com/watch?v=r27WcPRtpWM)

### Your First Boot ###
Once you are logged into your Raspberry Pi, open up the terminal and figure out what your IP is, this is going to be necessary to know for the rest of the tutorial:
- hostname -l

Once you get your Raspberry Pi setup and enable SSH, go ahead and run the following command to make sure everything is updated and ready for the rest of the installation.
- sudo apt-get update && sudo apt-get upgrade

### Can't Update Existing FoundryVTT to v9 ###
If you are already setup and can't update, run the following and make sure that you are running a Linux 11 OS.
- cat /etc/issue.net

Linux 10 and anything before cannot currently run FoundryVTT v9, so you will have to re-flash your system and update to the newest OS image. MAKE SURE YOU BACKUP EVERYTHING!

## Docker ##
Docker is very easy to setup and will essentially seperate it's container contents from the rest of the Raspberry Pi through virtualization. This will allow us to manipulate the network ports as we please for some added security as well as keep vital files and processes from overlapping with system processes.

To install Docker run the following:
- curl -sSL https://get.docker.com | sh

Give the current user admin rights for Docker:
- sudo usermod -aG docker ${USER}

To make any other users on the system and admin as well:
- sudo usermod -aG docker [user_name]

### Docker-Compose ###
Docker-Compose allows us to use a .yml file to keep our configuration, which allows us to change different parameters without the need to run a long line of commands.

To install this we need two other program functions, python and pip3, so run the following:
- sudo apt-get install libffi-dev libssl-dev
- sudo apt install python3-dev
- sudo apt-get install -y python3 python3-pip
- sudo pip3 install docker-compose

### Docker at Startup ###
We want to make sure that Docker runs anytime we restart our device.
- sudo systemctl enable docker

### Docker Install Reference ###
- https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo

## Portainer ##
If you want a nice easy to see GUI for all of your containers, you'll want to install Portainer. This is a container in Docker that lets you easily see everything running, give you other parameters to change in current containers, and you can view the logs of any containers to see whats going on.

Installation command:
- sudo docker pull portainer/portainer-ce:latest
- sudo docker pull portainer/portainer-ce:linux-arm (Just in case the latest image fails on the Raspberry Pi)

Now we can run the container:
- sudo docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest (or linux-arm)

Let's connect to Portainer using the Raspberry Pi's Hostname or IP address in any web broswer:
- http://[PIIPADDRESS]:9000

Create a new user, and make sure you select Docker for the environment that Portainer is using.

### Portainer Install Reference ###
- https://pimylifeup.com/raspberry-pi-portainer/

## Foundry VTT ##
Finally, lets get FoundryVTT setup and ready to run! We are going to make another docker-compose.yml, which you should make a foundry folder for so you don't lose it.
- sudo nano docker-compose.yml
- Place the following data into the newly created .yml, replacing all placeholder information with your own. The volumes > source is where you want your data to be on the raspberry pi, something like the following: /home/pi/foundry
```
---
version: "3.8"

services:
  foundry:
    image: felddy/foundryvtt:release
    hostname: my_foundry_host
    init: true
    restart: "unless-stopped"
    volumes:
      - type: bind
        source: <your_data_dir>
        target: /data
    environment:
      - FOUNDRY_PASSWORD=<your_password>
      - FOUNDRY_USERNAME=<your_username>
      - FOUNDRY_ADMIN_KEY=atropos
    ports:
      - target: 30000
        published: 30000
        protocol: tcp
   ```
   - If you need help figuring out what to add to the .yml, go to the creates [Github](https://github.com/felddy/foundryvtt-docker)
   - Once the .yml is created, lets start it up: docker-compose up -d --force-recreate

### Basic Troubleshooting ###
Make sure that your Linux version is 11 or later if you use the latest FoundryVTT image, otherwise it won't work. If you have Linux 10 or less, you will need this image:
- felddy/foundryvtt:0.8.9

Foundry username and password are for the website where you bought your key to install foundry.

### Foundry VTT Image REference ###

https://github.com/felddy/foundryvtt-docker

## Cloudflare Tunnels ##

### Getting A Domain ###
In order to self host, you're going to need a domain name so you can hide your public IP behind Cloudflare. [Freenom](https://www.freenom.com/en/index.html?lang=en) can provide you a free domain name of your choice, if it is avilable. The steps to getting it are simple enough and privded in the video. Thing to note Cloudflare now allows you to purchase Domain Names form them which is neat, and they are pretty cheap.

Once you have your domain name, you will need to change the name servers to the ones you get in your [Cloudflare](https://dash.cloudflare.com/login) account which you setup after adding your website.
- DNS Tab, scroll down
- ![image](https://user-images.githubusercontent.com/70184841/153463749-fd0db05f-d71c-4bf2-a019-c2c73f2c9a5c.png)
- Copy these name servers over the ones at the site you got your domain name from.

### Creating The Tunnel ###
Once the name servers have propogated and your website is in "Active Status" you''ll need to go to the "Access" tab and click on "Launch Zero Trust"

![image](https://user-images.githubusercontent.com/70184841/178129284-8b710f10-9e96-4a93-9397-b42bfc75e19b.png)

Now go to the "Access" tab again, and click "Tunnels"

![image](https://user-images.githubusercontent.com/70184841/178129303-cd20420e-1996-40ee-ad23-cff2f8079ca6.png)

Create a new tunnel

![image](https://user-images.githubusercontent.com/70184841/178129315-340d1fa4-a151-4171-af80-0a079208ca2f.png)
 
 Name your tunnel and save it, then you'll select an installer. Since we are using a Raspberry Pi in this tutorial and have the 64-bit OS installed, we choose "Debian > arm64-bit" but you can use whatever you want. The docker image didn't work with arm processors at the time of this recording just FYI!
 
 ![image](https://user-images.githubusercontent.com/70184841/178129370-2a36bfd0-9b1b-4728-a2ce-6d263d851b5a.png)
 
 All you have to do is copy and paste the box into your Raspberry Pi, it will begin the install of the tunnel and run it at the end.

Next you will be creating the subdomain if you want, select your domain name from the drop down, select HTTP, input the local network address of the Raspberry Pi, and click Save.

![image](https://user-images.githubusercontent.com/70184841/178129421-1b366d38-b335-4616-bde7-5524f1b6d03a.png)

You should get the tunnel up and running at this point and see the "Active" status.

![image](https://user-images.githubusercontent.com/70184841/178129454-1cf97b76-70cf-4b3c-b471-f9091ea4bda4.png)

You can then go into Configure for the tunnel, Public Hostnames, and see your link for the new domain of foundry you just created!

![image](https://user-images.githubusercontent.com/70184841/178129481-28892cd0-dd88-4573-923d-0496296c12ad.png)

### Application Secuirty (OPTIONAL)###

If you want to add those Access Lists I was atalking about, go to the "Access" tab and then "Applications" and click "Add an application".

![image](https://user-images.githubusercontent.com/70184841/178129540-10cc10ef-aa60-4d4f-b79a-3c0df1fdfe05.png)

Click on "Self Hosted"

![image](https://user-images.githubusercontent.com/70184841/178129558-9c487dbf-7633-416b-839f-910a2a805216.png)

Add in the information as needed, you can also edit the logo of the app for the Cloudflare Dashboard if you want, as well as add additional authentication paths in the future if you'd like. Otherwise the "One-Time Password" should already be selected.

![image](https://user-images.githubusercontent.com/70184841/178129576-e6715b94-f925-40fc-a8e1-1f212ced0c9f.png)

Now you will create a name for the policy, and torwards the bottom you can change the people that are allowed. If you use emails, make sure you get all the emails of the players and your own included in the values box. Any email that is not in that box will never get a one-time code.

![image](https://user-images.githubusercontent.com/70184841/178129661-280312c0-8edc-4291-a081-e46d53c98e57.png)

That's a pretty basic ACL, go ahead and tinker with it and add additional checks or methods of authentication as you see fit!

## Old Way ##
### Nginx Proxy Manager ###
Nginx Proxy Manager is a reverse proxy that allows us to only expose 2 ports on our modem/router instead of a bunch. In terms of security, you want as few ports forwarded on your modem/router since those forwarded ports allow all traffic regaurdless of its origin.

First lets make a folder to store all of the necessary files for this, you can place it in any directory, just keep in mind that this setup will place it in the /home/pi directory, pi being the current user running.
- mkdir nginx

Get into the new folder:
- cd nginx

Create a config file, and then put information into that config file:
- nano config.json

If you change any of the information besides the "changeme" portions keep it in mind for the rest of the install!
```console
{
  "database": {
    "engine": "mysql",
    "host": "db",
    "name": "npm",
    "user": "changeme",
    "password": "changeme",
    "port": 3306
  }
}
```
Save the new contents in the folder:
- ctrl + x then Y then Enter

### Compose File ###
Next while we are in the nginx folder, we need to make our .yml file that will hold all the docker container information and parameters. Make sure the information matches that of the config.json file we made before.
- nano docker-compose.yml

```console
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "changeme"
      DB_MYSQL_PASSWORD: "changeme"
      DB_MYSQL_NAME: "npm"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
  db:
    image: 'yobasystems/alpine-mariadb:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'changeme'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'changeme'
      MYSQL_PASSWORD: 'changeme'
    volumes:
      - ./data/mysql:/var/lib/mysql
```
There is a chance that the image we use for the db portion doesn't work depending on your OS, so you can try changing the "latest" to "10.4.17" instead.

### Deploying The Container ###
This is where everything comes together. The following command has to be ran in the same folder as our docker-compose.yml file, otherwise it won't do anything. When the command is ran, you'll see the system reach out to grab the images we specified in the .yml file, and fill those containers with all the parameters we specified. It should be noted that the --force-recreate doesn't do anything if there isn't a container already running, if there was a container running, it will take it down and remake it completely, which is useful for troubleshooting the .yml and trying new parameters.
- docker-compose up -d --force-recreate

### Connect To The Web GUI ###
Just like getting to Portainer, you'll connect to Nginx Proxy Manager using your hostname or IP and the port 81.
- http://[PIIPADDRESS]:81

The default credentials to login are as follows:
- Username: admin@example.com
- Password: changeme

### Port Forwarding ###
Now that we have Nginx Proxy Manager up and running, we need to forward all traffic on ports 80 and 443 (HTTP and HTTPS) to your Raspberry Pi running the container. Get into your modem/router, go to the port forwarding tab, and forward port 80 and 443 (TCP or BOTH) to the IP address of your Raspberry Pi.

### Nginx Proxy Manager Reference ###
- https://www.addictedtotech.net/nginx-proxy-manager-tutorial-raspberry-pi-4/

## Cloudflare & SSL ##
Cloudflare is my DNS service of choice, the free option adds a lot of security tools that can protect you and your now web hosted apps from people that wish to do harm. There are some key things that must be done in order for this to all work, so this might be the most important part of the guide.

### Getting A Domain ###
In order to self host, you're going to need a domain name so you can hide your public IP behind Cloudflare. [Freenom](https://www.freenom.com/en/index.html?lang=en) can provide you a free domain name of your choice, if it is avilable. The steps to getting it are simple enough and privded in the video. Thing to note Cloudflare now allows you to purchase Domain Names form them which is neat, and they are pretty cheap.

Once you have your domain name, you will need to change the name servers to the ones you get in your [Cloudflare](https://dash.cloudflare.com/login) account which you setup after adding your website.
- DNS Tab, scroll down
- ![image](https://user-images.githubusercontent.com/70184841/153463749-fd0db05f-d71c-4bf2-a019-c2c73f2c9a5c.png)
- Copy these name servers over the ones at the site you got your domain name from.

### A and CName Records ###
In the DNS tab of Cloudflare you will need to make a new A record, this will point the domain you have setup to your Public IP. Make sure it is Proxied as well.
![image](https://user-images.githubusercontent.com/70184841/153464511-0e8e362e-016c-4603-8b1c-1375c5bc3a9d.png)
Once you have the A record, we now need to make a subdomain CName Record for foundry itself. 
![image](https://user-images.githubusercontent.com/70184841/153464743-16c991cf-f00c-48b5-962c-985c2f772868.png)
- Note: This can be done as many times as you want, for as many different services or containers you are hosting.

### Security Settings ###
Now that Cloudflare is forwarding all traffic for your domain name to your public IP, we have to make sure the security settings are in place. Go to the SSL/TLS tab, and in the overview settins you will need to change to either Full or Full(Strict). Keep in mind that the Strict setting might slow down your instance just a little bit. Anything less most likely won't allow connections to your foundry.

![image](https://user-images.githubusercontent.com/70184841/153465509-4586159c-21b9-4d4a-9394-ee9a8abd8dc1.png)

### Creating An SSL Cert For Nginx ###
While we are in Cloudflare, lets go ahead and create our own certificate for SSL instead of using one from Let's Encrypt. These certs last for longer than Let's Encrypt. 
- In the SSL/TLS go to "Client Certificate" and click on "Create Certificate".
<img width="914" alt="image" src="https://user-images.githubusercontent.com/70184841/153466762-39c0dcfe-d6e4-425c-8926-bdb0ca9e0993.png">
- The next page we only really need to change the leangth, everything else meets our needs.
- Now we need to make 2 notepad files, copy and paste the contents of the boxes you see, and name them Certificate.com.pem (Certificate Text Box) and PrivateKey.com.key (Private Key Text Box)

![image](https://user-images.githubusercontent.com/70184841/153468303-6e693287-c20c-40ac-942f-b8e6e889ec9d.png)

- These files we will upload to Nginx Proxy Manager in the SSL tab so that we can use our cert for any services we create.

### Nginx SSL Cert Upload ###
Once you have your cert files download, head to your Nginx Proxy Manager, go the the SSL Certificates page, and create a new custom SSL Certificate.

![image](https://user-images.githubusercontent.com/70184841/153481708-ecbfa177-1190-4771-9f9c-dc65c099cefa.png)

Name your cert whatever, this is for Nginx only, and upload the appropriate files using the browse button.

![image](https://user-images.githubusercontent.com/70184841/153482040-b99fe35a-1a11-4628-ab4c-2b5e80a63d9d.png)

Now you should have an SSL cert for all of your subdomains to use through Nginx!


## Port-Forwading Workaround ##

DB Tech released a video a few months ago on how to use Cloudflare Tunnels to add more security to your self-hosting needs. I've started to implement it myself to test some things, and the neat part is it gets rid of the port forwarding requirement for Nginx and your router.

This not only allows you to protect your network and give you that ease of mind, but now for those of you that live in locations where you can't port forward, like a dorm or apartment with shared Wifi, you can use Cloudflare Tunnels and still self host! Really neat how it all works, so far I haven't found any issues with how my videos  go through the process, but I will make a new video sometime soon to go through a new installation process utilizing this method.

Link to the video:
https://www.youtube.com/watch?v=VrV0udRUi8A

I'll go through and screenshot the steps in the coming days and update this portion accordingly.
