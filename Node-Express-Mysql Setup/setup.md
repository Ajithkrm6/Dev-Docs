# Configuration:

1. First inisilaise the project using command `*npm init -y*`

2. Install all the necessary libraries like

   - Express by using command `npm install express`

   - Nodemon by using commabd `npm install nodemon` --> to run the server in dev mode so that we no need to retsarte the server all the time after making changes.

   - We have to configure in package.json

   ```
   "start" : "nodemon ./index.js" // replace with the file name
   ```

   - visit **https://www.npmjs.com/** to install third party libraries.

   - search **colors** and install the module by given command.

   - search **morgan** and install the module by given command, with this we can view the endpoint which is requested in the console.

   - search **mysql2** and install to make db connection.

   - search dotenv and install, by this approch we can hide confidential details
     `dotenv.config()`

   - once after installing create file in root directory with name **.env**, next in app.js or index.js configure the dot env my importing,

   ```
   const dotenv = require('dotenv');
   dotnev.config();
   ```

   when we make changes in .env we have to **_restart the server_**

   The package.jon must look like:

   ```
   {
   "name": "backendtest",
   "version": "0.0.0",
   "private": true,
   "scripts": {
    "start": "node ./bin/www",
    "server": "nodemon app.js"
   },
   "dependencies": {
    "colors": "^1.4.0",
    "cookie-parser": "~1.4.4",
    "debug": "~2.6.9",
    "dotenv": "^16.4.5",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "jade": "~1.11.0",
    "morgan": "~1.9.1",
    "mysql2": "^3.11.0",
    "nodemon": "^3.1.4"
   }
   }
   ```

3. In the terminal use command `npx express-generator  or  npm i express-generator` with this command we can create the recommended project skeleton.

# Folder Structure:

- controllers --> All the business logic will be there in the controllers

  - userController.ts

- routes --> all the routes will be created in folder for example
  - login.js ```ex:router.get("/create-user",createuser)
  - signup.js
  - users.js

```

```
