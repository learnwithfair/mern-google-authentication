# MERN-GOOGLE-AUTHENTICATION

[![Youtube][youtube-shield]][youtube-url]
[![Facebook][facebook-shield]][facebook-url]
[![Instagram][instagram-shield]][instagram-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

Thanks for visiting my GitHub account!

<img src ="https://miro.medium.com/v2/resize:fit:548/1*KEtWAhxgLOKU8Sd1np4Phg.png" height = "200px" width = "200px"/>**Sign in with Google** makes it easy for you to sign in and sign up to websites and apps across the internet with the trusted security of your Google Account. It eliminates your dependency on passwords, which reduces the frustrations and security risks associated with them. [see-more](https://blog.logrocket.com/guide-adding-google-login-react-app/)

### [Code-Example](https://github.com/learnwithfair/mern-user-admin-mgt)

## Source Code (Download)

[Click Here]()


## Required Software (Download)

- VS Code, Download ->https://code.visualstudio.com/download
- Node, Download-> https://nodejs.org/en/download
- MongoDB Shell(msi) , Download-> https://www.mongodb.com/try/download/shell
- MongoDB Compass (msi), Download-> https://www.mongodb.com/try/download/community
- Postman, Download-> https://www.postman.com/downloads/

**Or Online Database (MongoDB Atlas)**

- Register -> https://www.mongodb.com/cloud/atlas/register

## ========== Environment Setup ==========

1. Install Node.js
2. To verify installation into command form by node -v
3. For initialization npm write the query in the command window as npm init -y
4. Setup the opening file into the package.json and change the file with main:'server.js'
5. To create a server using the express package then write a query into the command window as npm install express.
   Write code in the server file for initialization
   const express = require("express");
   const app = express();
   app.listen(3000, () => {
   console.log("Server is running at http://localhost:3000");
   });

6. Install the nodemon package for automatically running the server as- npm i --save-dev nodemon (For Developing purpose)
7. setup the package.json file in the scripts key, write
   "scripts": {
   "start": "node ./resources/backend/server.js",
   "dev": "nodemon ./resources/backend/server.js",
   "test": "echo \"Error: no test specified\" && exit 1"
   },
8. use the Morgan package for automatic restart. Hence install the morgan package as npm install --save-dev morgan (Development purpose)
   Write code in the server file for initialization
   const morgan = require("morgan");
   app.use(morgan("dev")); --> Middlewire.
9. Install Postman software for API testing by the URL endpoint.
10. Install Mongobd + MongobdCompass and Mongoshell (For Database)

## ========== Connect MongoDB Database ==========

1. Install Mondodb + Mongodb Compass and Mongodb Shell download from the google.
2. Set up Environment Variable in drive:c/program file
3. Create a directory in the base path of the c drive named data. Inside the data directory create another folder db.
4. Write the command in the CMD window as Mongod. And write the other command in the other CMD window as mongosh.
5. Then Check the version as mongod --version and mongosh --version.
6. Install mongoose package as npm i mongoose
7. Create an atlas account. In the atlas account create a cluster that have a user(as atlas admin) and network access with any access IP address.
8. Connect the database using URL from the atlas cluster or local Mongodb compass using the mongoose package as mongoose. connect('mongodb://localhost:27017/database-name);

## How to use this template

- Step-1: In the Frontend site

```js
import React, { useEffect } from "react";
import GoogleLogin from "react-google-login";
import { gapi } from "gapi-script";
import { loginWithGoogle } from "../services/AuthService";

const Google = () => {
  const responseGoogle = async (response) => {
    try {
      // 1. get the tokenId from response
      console.log(response.tokenId);

      // 2. send the tokenId to the server
      const result = await loginWithGoogle({ idToken: response.tokenId });
      console.log("Google signin success: ", result);
    } catch (error) {
      console.log("Google signin error: ", error.response.data.message);
    }
  };

  useEffect(() => {
    function start() {
      gapi.client.init({
        clientId: process.env.REACT_APP_CLIENT_ID,
        scope: "email",
      });
    }

    gapi.load("client:auth2", start);
  }, []);

  return (
    <div className="pb-3">
      <GoogleLogin
        clientId={`${process.env.REACT_APP_CLIENT_ID}`}
        render={(renderProps) => (
          <button
            onClick={renderProps.onClick}
            disabled={renderProps.disabled}
            className="btn btn-danger btn-lg btn-block"
          >
            <i className="fab fa-google p-2"></i>Login With Google
          </button>
        )}
        onSuccess={responseGoogle}
        onFailure={responseGoogle}
        cookiePolicy={"single_host_origin"}
      />
    </div>
  );
};

export default Google;
```

- Step-2: In the `services/AuthService` path

```js
export const loginWithGoogle = async (data) => {
  const response = await axios.post(
    `http://localhost:3030/api/google-login`,
    data
  );
  return response.data;
};
```

- Step-3: In the Server site

  - install `npm install google-auth-library`

```js
// google
authRoutes.post("/google-login", handleGoogleLogin);

const { OAuth2Client } = require("google-auth-library");
const jwt = require("jsonwebtoken");

const handleGoogleLogin = async (req, res) => {
  try {
    // creating client which help us to verify the idToken that we receive from front end
    const client = new OAuth2Client(dev.app.googleClientId);

    // get the idToken form react app request body
    const { idToken } = req.body;
    console.log("idToken ", idToken);

    // lets verify the idToken
    client
      .verifyIdToken({ idToken, audience: dev.app.googleClientId })
      .then(async (response) => {
        console.log("GOOGLE LOGIN RESPONSE", response);
        const { email_verified, name, email } = response.payload;

        const exsitingUser = await User.findOne({ email });
        if (email_verified) {
          // if the user already exist in our website
          if (exsitingUser) {
            console.log("user exist");
            // create a token for user
            const token = jwt.sign(
              { _id: exsitingUser._id },
              String(dev.app.jwtSecretKey),
              {
                expiresIn: "7d",
              }
            );

            // step 7: create user info
            const userInfo = {
              _id: exsitingUser._id,
              name: exsitingUser.name,
              email: exsitingUser.email,
              phone: exsitingUser.phone,
              isAdmin: exsitingUser.isAdmin,
            };

            return res.json({ token, userInfo });
          } else {
            // if user does not exist create a new user
            let password = email + dev.app.jwtSecretKey;
            const newUser = new User({
              name,
              email,
              password,
            });

            // console.log(newUser);

            const userData = await newUser.save();
            if (!userData) {
              return res.status(400).send({
                message: "user was not created with google",
              });
            }

            // if user is created
            const token = jwt.sign(
              { _id: userData._id },
              String(dev.app.jwtSecretKey),
              {
                expiresIn: "7d",
              }
            );

            // step 7: create user info
            const userInfo = {
              _id: userData._id,
              name: userData.name,
              email: userData.email,
              phone: userData.phone,
              isAdmin: userData.isAdmin,
            };

            return res.json({ token, userInfo });
          }
        } else {
          return res.status(400).send({
            message: "Google login failed try again",
          });
        }
      });
  } catch (error) {
    res.send({
      message: error.message,
    });
  }
};
```

- Step-4: Setup .env file in the server folder

```env
SERVER_PORT=8080
MONGODB_URL=
JWT_ACCOUNT_ACTIVATION_KEY=
JWT_RESET_PASSWORD_KEY=
JWT_ACCESS_TOKEN_KEY=
JWT_REFRESH_TOKEN_KEY=
SMTP_USERNAME=YOUR_GMAIL_HERE
SMTP_PASSWORD=
CLIENT_URL=
SESSION_SECRET=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

```

## Follow Me

[<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/github.svg' alt='github' height='40'>](https://github.com/learnwithfair) [<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/facebook.svg' alt='facebook' height='40'>](https://www.facebook.com/learnwithfair/) [<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/instagram.svg' alt='instagram' height='40'>](https://www.instagram.com/learnwithfair/) [<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/twitter.svg' alt='twitter' height='40'>](https://www.twiter.com/learnwithfair/) [<img src='https://cdn.jsdelivr.net/npm/simple-icons@3.0.1/icons/youtube.svg' alt='YouTube' height='40'>](https://www.youtube.com/@learnwithfair)

<!-- MARKDOWN LINKS & IMAGES -->

[youtube-shield]: https://img.shields.io/badge/-Youtube-black.svg?style=flat-square&logo=youtube&color=555&logoColor=white
[youtube-url]: https://youtube.com/@learnwithfair
[facebook-shield]: https://img.shields.io/badge/-Facebook-black.svg?style=flat-square&logo=facebook&color=555&logoColor=white
[facebook-url]: https://facebook.com/learnwithfair
[instagram-shield]: https://img.shields.io/badge/-Instagram-black.svg?style=flat-square&logo=instagram&color=555&logoColor=white
[instagram-url]: https://instagram.com/learnwithfair
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=flat-square&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/company/learnwithfair
