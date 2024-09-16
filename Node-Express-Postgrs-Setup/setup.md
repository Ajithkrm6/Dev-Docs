1. Install
   - Express
   - pg
   - nodemon
   - corse,
   - morgan
   - typescript
   - dotenv,
   - multer
   - bcrypt
   - jsonwebtoken
   - path

```
npm install express pg nodemon corse morgan
npm install typescript ts-node tsconfig-paths @types/node --save-dev
or
npm install --save-dev typescript @types/node @types/express ts-node-dev
```

2. Setup tsconfig
   create a tscongig file

```
npx tsc --init
```

3. In the tsconfig.json, make sure to configure it like this for typical Express setups:

```
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "exclude": ["node_modules"]
}

or

{
  "compilerOptions": {
    "target": "ES6", // Target JavaScript version
    "module": "commonjs", // Module system
    "strict": true, // Enable all strict type checking options
    "esModuleInterop": true, // Enables emit interoperability between CommonJS and ES Modules
    "skipLibCheck": true, // Skip type checking of declaration files
    "outDir": "./dist", // Output directory for compiled files
    "rootDir": "..", // Root directory of the source files
    "typeRoots": ["./node_modules/@types"], // Make sure this is included
    "types": ["node", "express", "bcrypt"] // Optional but can be helpful
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}

```

4. To start the server add script in package.json

```
"scripts":{
   "test": "echo \"Error: no test specified\" && exit 1",
    "server": "nodemon --exec ts-node -r tsconfig-paths/register src/index.ts "  // this is main, as this line starts the server
}
```

5. Database Configuration:
