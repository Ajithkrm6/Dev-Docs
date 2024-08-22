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

3. In the terminal use command `npm install express-generator` with this command we can create the recommended project skeleton.
