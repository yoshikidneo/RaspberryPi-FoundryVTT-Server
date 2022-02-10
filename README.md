# RaspberryPi-FoundryVTT-Server
![image](https://user-images.githubusercontent.com/70184841/153284283-aa073c52-326a-4871-9646-8ef87afc0aaf.png)

This is the step by step for my video, with links to other videos, hopefully this makes things easier especially for copy and pasting!

## Raspberry Pi Setup ##
First things first, setup your Raspberry Pi with any OS you wish, I was using Twister OS but have switched to the latest Raspbein 32bit since it is running on Linux 11, which is necessary for FoundryVTT v9 or later. 
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

## Nginx Proxy Manager ##
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
![image](https://user-images.githubusercontent.com/70184841/153467135-8c7cfc44-6574-4886-b9c4-2e3b3c30ea2b.png)
- These files we will upload to Nginx Proxy Manager in the SSL tab so that we can use our cert for any services we create.




## Foundry VTT ##

https://github.com/felddy/foundryvtt-docker
