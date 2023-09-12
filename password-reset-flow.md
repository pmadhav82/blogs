## A Deep Dive into Password Reset Flows with Node.js and Express.js
![blog-image](https://images2.imgbox.com/0f/2b/imkAL9B7_o.png)
When it comes to web development and ensuring the security of user accounts, the password reset process is a critical component that should not be overlooked. Therefore, implementing a robust and secure password reset flow is a necessity for any web application that values user security and usability.

In this blog post, we will learn how to implement a robust password reset flow in your Node.js (Express.js) application. By the end, you'll have a comprehensive understanding of how to savegaurd your users's accounts and provide them with seamless password reset experience.

Below is the password reset flow diagram:

![flow-diagram](https://images2.imgbox.com/4e/ca/2H39Mvzm_o.png)

Now, let's breakdown what sptes are involved in this process one by one:
1. Request for password reset: use can send request for password reset by filling password reset form
2. Valid user information: The system will validate user exist or not
3. Send an email to the user with token: If user is exist, the system will send an email to provided email ID with user id and a token which will have within certain time
4. Check the token is valid or expired: When user clicked on sent link, the system will again verify if the token is valid or not and if the token is related to that particular user or not
5. Set the new password: If everything goes well, finally user will be given a password reset form where they can enter new password



 1. Request for password reset
 Once user clicked on `Forgot password` link the server will get `get` request and send a form where user can enter their email id. The following code goes in `router.js` file.

 ```
 //password reset route
router.get("/forgot-pass", ifLogin, (req,res)=>{
    res.render("reset")
 })
 
 ```
Now, let's create `reset.handlebars` file where we will have form

```
<div class="login-box">
 <div class="messages">
  {{>message}}
 </div>
 
  <form action="/password-reset" method="POST">
  <h2>Reset your password</h2>

    <div class="user-box">
     
      <input type="text" name="email" value="{{email}}" required>
      <label>Email address</label>
    </div>
<button type="submit">Send link</button>

  </form>

</div>
```
![password-reset-form](https://images2.imgbox.com/59/51/GXHushOQ_o.png)


Once, the user enter thier email id and hit `Send link` button the server will get `post` request at `/password-reset` endpoint.

2. Validating user information an sending email with link

This is the most time taking part of this process. Where we have to verify user information, setup `nodemailer`, generate a token and save token in database plus send an email with token and user id. This all process happen when server receive `post` requeset from `/password-reset` endpoint

First, let's create `tokenSchema` in `token.js` file to save token with user information and expire time of 1 hour.

```
const mongoose = require("mongoose");
const{Schema, model}= mongoose;

const tokenSchema= new Schema({
    userId:{
        type:Schema.Types.ObjectId,
        require:true,
        ref:"Users"   
    },
    token:{
        type:String,
        require:true
    },
    createdAt:{
        type:Date,
        default:Date.now,
        expires:3600
    }
})

module.exports = new model("token",tokenSchema);
```
Now, let's create another file `tokenHandeler.js` file. Where we will write generating token, validating token and saving the token in database logic


```
const Token = require("../module/token");
const bcrypt = require("bcrypt");
const crypto = require("crypto");


const generateToken = async(id)=>{
let token = await Token.findOne({userId:id});
// delete already exits token
if(token){
    await Token.deleteOne({userId:id});
}

// generate token
const resetToken = crypto.randomBytes(32).toString("hex");

//hash reset token
const hashedToken =  await bcrypt.hash(resetToken,10)

// save token in database

await new Token({
    userId:id,
    token:hashedToken,
    createdAt:Date.now()
}).save()

return resetToken;

}



const isValidToken = async({token,id})=>{
    try{
        const savedToken = await Token.findOne({userId:id}).lean();
        //compare the token
        if(savedToken){
            const isValidToken = await bcrypt.compare(token,savedToken.token);
            return isValidToken;
        }else return false

    } catch(er){
        console.log("something went wrong..." )
    }

}



module.exports = {
    generateToken,
    isValidToken
}

```
In the above code, we have `generateToken` function which will generate unique token with the help of `crypto` package. We hashed that token and save it to database along with user id and we have `isValidToken` function which will take `token` and `id` and compare it with saved `token` and `userID`

Let's setup `nodemailer`. For that create a file `sendEmail.js`. The file will have following code:

```
const nodemailer = require("nodemailer");

const sendEmail = (payload)=>{
const { email, subject, html}= payload

const transporter = nodemailer.createTransport({
    host:process.env.EMAIL_HOST,
    secure:true,
    port:465,
    auth:{
        user:process.env.EMAIL_USERNAME,
        pass:process.env.EMAIL_PASSWORD
    }
})

const mailOptions ={
from:"p.blog@pblog.online",
to:email,
subject:subject,
html:  html
}

transporter.sendMail(mailOptions,(error,info)=>{
    if(error){
        console.log(error)
    }else{
console.log(info);
    }
    
})

}

module.exports = sendEmail;
```
In the above code, we have `sendEmail` function. Inside that function we have setup `nodemailer` and using this function we can send an email to the user with the password reset link.

I have created a **Zoho Mail Account**. In my case `host` is `smtp.zoho.com.au`, `user` will be my Zoho mail and `pass` is zoho application password.

Since we have setup `nodemailer` and `token`, now we are ready to handle `post` request came from `/password-reset`. In `router.js` file:

```
const express = require ("express");
const router = express.Router();
const sendEmail = require("./utils/sendEmail");
const {generateToken, isValidToken} = require ("./utils/tokenHandeler");
const flash = require("connect-flash");

// middleware fucntion to associate connect-flash on response
router.use((req,res,next)=>{
res.locals.message = req.flash();
next()
})

// password reset post route

router.post("/passport-reset", async (req,res)=>{
    const {email} = req.body;
    try{
let user = await Users.findOne({email});

if(user){
    const resetToken =  await generateToken(user._id);
    const link= `${req.protocol}://${req.get('host')}/password-reset-link?token=${resetToken}&id=${user._id}`;
 
// html for email
const html = `<b> Hi ${user.name}, </b>
<p> You requested to reset your password. </p>
<p> Please, click the link below to reset your password. </p>
<a href = "${link}"> Reset Password </a>
`

console.log(link);

const payload = {
    email,
    subject:"Password reset request",
    html
}

sendEmail(payload);


req.flash("success", "Check your email for the password reset link")
    res.redirect("/login")
}else{
    req.flash("error","We could not find any user, please check your email address again")
res.redirect("/forgot-pass")
}
    }catch(er){
        console.log(er);
        req.flash("error","Something went wrong, please try again later!")
        res.redirect("/forgot-pass")
    }
})

```
Let's break it down above code. First, we looked for user associated with an email id provided by user. If user is not found then we have redirect user to `/forgot-pass` route with an `error` message. If user is found we generate a `token` using `generateToken` function and we created a dynamic link where we have `token` and `user_id` in the link. We send an email to the user by using `sendEmail` function. Then, we have redirected user to `/login` router with `success` message.

![sent-reset-link](https://images2.imgbox.com/b8/a0/eVquMoNc_o.png)

The password reset link will look like this:
```
http://localhost:8000/password-reset-link?token=e51d95cd5feba716fbe54163275ff874795fe7b52a625404a7e5adf00d3b2c22&id=62cd89d53e61de6c5bfbfe02

```
On success, user will get an email where they can click to get the form where they can enter new password and send it back to server.

![reset-link-email](https://images2.imgbox.com/64/de/FTMi6KUL_o.png)

I have used `connect-flash` to send message whenever a user is redirecting to specified route. I have created `message.handlebars` file inside `partial` folder and following code is in that file:

```
  {{#if message.success}}
  <div class="messages success-message">
      {{message.success}}
  </div>
  
    {{/if }}

   {{#if message.error}}
  <div class="messages error-message">
      {{message.error}}
  </div>
    {{/if }}
```
This will show any error or success message to the user.

 3. Validating the token and sending password-reset form
When user click on that reset link, server will get `get` request on `/password-reset-link`. Let's handle this route together :)

```
//password reset form route
router.get("/password-reset-link",  async (req,res)=>{

if(req.query && req.query.token && req.query.id){
    //check token and id are valid
const{token,id} = req.query;

try{
   const isValid = await isValidToken({token,id});
    if(isValid){
res.render("newPasswordForm",{
    token,
    id,

})
    }else{
res.json({message:"Invalid token or link is expired"})
    }

}catch(er){
    console.log(er)
res.json({message:"something went wrong, please try again latter"})
}


}else{
    res.redirect("/login")
}

})

```
In the above code, we have extracted the `token` and `user_id` from the link. We validate `token` and `user_id` with the help of `isValidToken` function that we have created in `tokenHandeler.js` file. If the link is valid `newPasswordForm` is rendered and we have sent  `token` and `user_id` as well. These are needed again when we get `post` request, once user enter new password and submit the form.

One important thing to remember here is that user need to click on that link within 1hour as our token expires in 1hour. 
If user tried to modify the `URL` or click the link after 1hour, user would not be able to see the form insted user will see an error message

![error-message](https://images2.imgbox.com/79/47/Zd89v9Wx_o.png)

Now, let's create a file `newPasswordForm.handlebars` file inside `views` folder.

```

  <form action="/newPassword?token={{token}}&id={{id}}" method="POST">  
  <h2>Enter new password</h2>

<div class="user-box">
<input  type="password" name="password"  required>
<label > Password </label>
</div>

<div class="user-box">
<input type="password" name="repeatPassword"  required>
<label > Repeat Password </label>
</div>
<button type="submit"> Change Password</button>
 
  </form>

```
This form will only shows up if the link is valid.

 ![new-password-form](https://images2.imgbox.com/a4/85/37u86Znn_o.png)
