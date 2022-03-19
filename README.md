
# Express Node.js servers on Synology NAS 

<br />

![Capture d’écran 2022-03-19 154906](https://user-images.githubusercontent.com/91158824/159125852-4125aa7a-4e72-446f-8125-fb8ba55dec11.jpg) 


<br />



📌 This tutorial is a short mix of these 2 documents https://github.com/StephanThierry/nodejs4synologynas and https://timonweb.com/javascript/running-expressjs-server-over-https/ 

Don't hesitate to tell me if you spot any errors or you want to add improvements.

<br />
<br />
 
### Before everything

Create a new non-encrypted shared folder in synology, "server" (or anything you want).
In "server" create a new folder for your express server, "myServer" for example and create an index.js in it with the code below : 

```js
var express = require("express");

var fs = require("fs");

var https = require("https");

var app = express();

  

app.get("/", function (req, res) {

 res.send("hello world !");

});

  

https

 .createServer(

 {

 key: fs.readFileSync("server.key"),

 cert: fs.readFileSync("server.cert"),

 },

 app

 )

 .listen(4562, "192.168.1.65", function () {

 console.log(

 );

 });
 ```

Here the port chosen is "4562", for a synology nas with the following local adress "192.168.1.65".

<br />
<br />


### Then it is necessary for our express server to have ports opened on our synology nas  

In synology control panel>External access>router configuration>create>custom port, put the port you wrote in your express server, "4562" here.

Go to Security>Firewall>edit rules>create>put custom port "4562"  for all ip (you can try precise limitations)

<br />
<br />


### We need to activate ssh control of the synology nas 

Control panel>Terminal & SNMP>enable SSH service, change the port number, don't keep "22" to avoid obvious hacks. Here we decide to use "345".

Go to security>firewall>edit rules>create>ports>select from a list of buil-in applications>encrypted terminal service for all IP (you can try limitations but for me it was necessary)

External access>router configuration, if the Encrypted terminal service line is not here, clic create>built-in application>Encrypted terminal service>apply.

<br />
<br />


### Login with ssh tool 

Putty or another one, but I chose Putty because windows powerShell refused to work.

To get in we need to have the id and password of a synology user who's part of the administrators group.

In Putty, type the name of the website that leads to your synology, for example "myWebsite.no-ip.com" without "http" or "https", and the port number you wrote in Terminal & SNMP, "345".

Then type your id and password in the terminal when it is asked

In the terminal go to your server's folder volume1/server/myServer, enter the commands below :
<br />
`npm init --yes` 
<br />

`npm install express --save`

Create a self signed certificate using `openssl req -nodes -new -x509 -keyout server.key -out server.cert`

<br />

 📌You'll get a warning from your browser when visiting your server page at "myWebsite.no-ip.com:4562" because the self signed certificate is not enough. You can do the process for Let's encrypt using certbot, but the process is longer. 
 
<br />

To launch the server with your terminal type `node .`

To visit your server type "myWebsite.no-ip.com:4562" and the server will answer

If we want the server to run even if we closed the terminal window, we must install forever with `sudo npm install forever -g` then launch it  `sudo forever start index.js`  

This process seem to work with 2 simultaneous servers running, I haven't tried on more.

We can close our ssh tool and the server will continue to run.

If we want to stop later our server, go back at his location with your ssh tool volume1/server/myServer and type  `sudo forever stopall` 

<br />

📌For safety reasons it is advised to close the ssh when the server is launched  and all the ports linked to it in the security>firewall and external access>router configuration
