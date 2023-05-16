# File Upload using multer in ExpressJS
`multer` is a middleware of NodeJS to handle file upload.
## Instalation
``` npm install multer ```

## Frontend
```
<div class="form-wrapper">
<h3>Upload Profile Picture</h3>

<div class="main">
<form action="/upload" method="POST" enctype="multipart/form-data">

<input type="file" accept="image/*" id="imageInput"  name="userProfile" />
<button type="submit" id="submit" class="btn btn-primary">Upload</button>
</form>

<div class="image-preview">
<img id="imageOutPut"  />
<p id="imageName"></p>
</div>

</div>

```
### Upload preview
Following code will enable the preview of choosen image
```javascript
    
let imageInput = document.getElementById("imageInput");
let imageOutput = document.getElementById("imageOutPut");
let imageName = document.getElementById("imageName");
imageInput.onchange = (ev)=>{
    imageOutput.alt = "preview";
    imageOutput.src = URL.createObjectURL(ev.target.files[0]);
    imageName.innerHTML = `<b> ${ev.target.files[0].name} </b>`
    imageOutput.onload = ()=>{
        URL.revokeObjectURL(imageOutput.src);
    }
}

```

![preview](https://images2.imgbox.com/07/f7/ah9CzDGQ_o.png)


## Backend
We have to setup basic  `multer` configuration which includes where do we want to save uploaded image, what name do we want to set up. We can filter the file as well and we can set the file size limit.
``` javascript
//fineName: fileUpload.js

 const multer = require("multer");
const path =  require("path");

const upload = multer({
    limits:800000,
    storage: multer.diskStorage({
destination:(req,file,cb)=>{
cb(null,"upload/images")
},
filename:(req,file,cb)=>{
    let ext = path.extname(file.originalname);
cb(null,`${someUniqueName}.${ext}`)
}
    }),

    fileFilter:(req,file,cb)=>{
        const allowedFileType = ["jpg", "jpeg", "png"];
        if(allowedFileType.includes(file.mimetype.split("/")[1])){
            cb(null,true)
        }else{
            cb(null,false)
        }
    }

})
module.exports = upload;

```
### Explationation of above code
`multer` take 3 objects `limits`, `storage` and `fileFilter` as a parameter. `storage`have 2 parameter which are `diskStorage` and give full control on storing file to disk and `filename`. 

Now, we can export this as a middleware and we can use to upload image as follow:
## Route setup to handle uploaded file

As our frontend form will hit the endpoint `/upload` we have to setup our route to handle this endpoint which has image in it plus we need to `require` the middleware that we created.

``` javascript
//fileName: route.js

const express = require ("express");
const router = express.Router();
const upload = require("./fileUpload");


router.post("/upload/",  upload.single("userProfile"),(req,res)=>{
   
    if(!req.file){
        return res.redirect("/")
    }
    else{   
        
        let filePath = `images/${req.file.filename}`;
       res.render("image",{
        filePath
       }) 
    }
})

```
`upload.single()` will handle single image came from request and saved it to specified folder. The image is saved to `upload/images` folder.
Now, we have to make `upload` folder static so that images saved there can be served to the frontend and file path will be  `images/${req.file.filename}` we can save this `imageURL` to the database as well.

``` javascript
//fileName:app.js

const route = require("./route");
const app = express();
const PORT = process.env.PORT || 8000

app.use(express.urlencoded({extended:true}))


//router connection
app.use("/", route);



//uses of public folder

app.use(express.static(path.join(__dirname,"/public")))
//app.use(express.static(path.join(__dirname,"/upload")))
app.use(express.static(`${__dirname}/upload`))


//init handlebars
app.engine("handlebars", handlebars());

app.set("view engine", "handlebars");


app.listen(PORT,()=>console.log(`Server is running on ${PORT}`))


```
This way you can upload image to the server using `Expressjs` with the help of `multer`.

