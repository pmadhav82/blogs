# How to deploy nodejs app on AWS EC2

1. Create ec2 instalce by choosing linux 2

2. Leave everything default and click launch instance

3. Go to instances and click your instace and click on connect.
 Then, you will see option to connect to instance. If you want to access instance terminal from web-browser you can click on  `Connect` button of `EC2 Instance Connect`. Or, you can connect through SSH client. For that click on `SSH client` tab to see what steps to take.
 As per instructions provided here, you need to open `SSH client` such as linux terminal or git bash.
 Go to the folder where private key `.pem` file is located and run following command which will give read only permision to the file

 ``` 
 chmod 400 prevate-key-name.pem
 ```

 Then, you are ready to connect instance using its public DNS or public ip address by following command




```
ssh -i "node-server-key.pem" ec2-user@13.54.206.174
```

Here, private key name is `node-server-key`, user is `ec2-user` and public id address is `13.54.206.174`. You can use public DNS as well in place of public address
```
ssh -i "node-server-key.pem" ec2-user@ec2-13-54-206-174.ap-southeast-2.compute.amazonaws.com             
```

After that you can have access to terminal through SSH client. Once you have access to the terminal,type following command

``` 
sudo yum install -y gcc-c++ make
```

then, it install nodejs by typing following command
 Type following command

```
sudo yum install -y nodejs 
```

 To check the version of nodejs run
`node -v` command

4. Now you can clone node app from your github. Once the repo is downloaded go to that folder and run 
`npm instal`
 which will install all the dependency. For environmental variables, you can create a new directory and create a `.env` file there. You can put all environmental variables in that file:
```
 touch /var/app/env/.env
 ```
Edit the file by running `nano /var/app/env/.env`
 Save the `.env` file and we need `dotenv` package to load those environmental variables. For that run following command

 ```
 npm install dotenv
 ```
 Once it installed, in node.js application, we need config `dotenv` package on top of main application file.

 ```
require('dotenv').config({path: 'var/app/env/.env'})
 ```
For the access type your `publicIP:portNum`. In my case it is `http://13.54.206.174:3000/`


5. If you want to run your sever in the background you can you `pm2` which is a process manager for node.js application. `pm2` can handle to start, restart and stop the server and log managmement. To install `pm2` run following command
```
npm install pm2 -g
```
To start the server use command:

```
pm2 start app.js
```
To verify your server is running run following command

```
pm2 list
```
This will display a list of all processes managed by `pm2`

To stop the server
```
pm2 stop app.js
```

6. If you have a domain name and you want to use it, you need to install `nginx` which is a web server by following command

```
sudo yum install nginx
```
Once it installed, you need to configure Nginx to act as a reverse proxy for the application. Open the configuration file by following command

```
sudo nano /etc/nginx/nginx.conf
```

In the file in server blog add your domain name

```
server {
     listen       443 ssl http2;
       listen       [::]:443 ssl http2;
       server_name YOURDOMAIN_NAME;
       location = {
proxy_pass http://localhost:3000;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
       }
}
```