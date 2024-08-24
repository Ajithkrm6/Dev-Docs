# App.js:---

```
// imports
const express = require("express");
const colors = require("colors);
const morgan = require("morgan);
const dotenv = require("dotenv");

// configure dotenv
dotenv.config() // if the .env file is in root and app.js is also in root no need to give path, else we have to mention the path as
// dotenv.config({path:"./example.js"})

// create a rest object
const app = express();

// middleware
app.use(morgan("dev"));
app.use(express.json()); // by this json can be accepcted, if in request.body has json else it will not be expected.

// routes
app.get("/test",(req,res)=>{
res.status(200).send("<h1>Working on node + express + mysql");
})

// port
const PORT =process.env.PORT || 8080;

//listen
app.listen(PORT,()=>{
    console.log(`server is running on port ${PORT}`.bgMagenta.white)
})

```
