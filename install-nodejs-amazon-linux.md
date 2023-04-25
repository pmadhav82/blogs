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
// here your private key name is node-server-key and user is ec2-user and public id address is 13.54.206.174

//you can use public DNS as well in place of public address

ssh -i "node-server-key.pem" ec2-user@ec2-13-54-206-174.ap-southeast-2.compute.amazonaws.com

```

After that you can have access to terminal through SSH client. Once you have access to the terminal,type following command
``` 
sudo yum install -y gcc-c++ make
```
then, it install nodejs by typing following command
 Type following command
```sudo yum install -y nodejs ```

 To check the version of nodejs run
`node -v` command

4. Now you can clone node app from your github. Once the repo is downloaded go to that folder and run 
`npm instal`
 which will install all the dependency.
For the access type your `publicIP:portNum`. In my case it is `http://13.54.206.174:3000/`








