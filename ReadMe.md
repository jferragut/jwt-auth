# User Authentication with JWT
by Jonathan J. Ferragut

**Technologies:** Wordpress, JWT, Javascript, ReactJS
_____

When developing headless Wordpress websites with React, we will often times need to set up Users for our site. This means private routes/requests, and the need to authenticate said requests.

In the following sections, we will discuss some caveats and scenarios that will help you to better understand the workflow of setting up JWT on Wordpress and subsequently dealing with requests in React using Fetch API.

This document assumes that you have:
 1. Installed and configured a Wordpress install already
 2. Have a Frontend developed in ReactJS
 3. Are familiar with HTTP Requests

**Note:** All routes on the Wordpress API will start with `https://yourdomain.com/wp-json/`, where `yourdomain.com` should be replaced with the domain/host that you are using for your server. If you are using Gitpod, this will be a workspace URL that is assigned randomly to your instance. This will look something like:  `https://ffe4c36b-7ff9-4057-ba10-634bd14f8182.ws-us0.gitpod.io`

# JWT Plugin

The journey begins with the setup and install of JWT on your Wordpress backend. We'll discuss a basic installation procedure, but if you require more, you can defer to the plugin documentation.

Please make sure to Install the JWT Authentication plugin which can be found here: [JWT Plugin Download Page](https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/)

## How to setup JWT on Wordpress

Setting up the JWT plugin starts with modifying the `.htaccess` file, which controls settings for the apache server. After that, you will need to add a configuration value to the `wp-config.php` file.

**Step 1:** 
Open the `.htaccess` file and look for the line that says `RewriteEngine On`.  Immediately after this line, insert the following two lines of code:

```
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]
```

This will enable authorization on routes and require that you use Bearer Tokens access secure routes. *Please make sure to save your changes.*

***Note:*** *If you encounter a 503 error with your fetch on the frontend , check to see if the settings in your `.htaccess` file have been overwritten.*

**Step 2:**  
Open the `wp-config.php` file and search for a line that says `define( 'WP_DEBUG', true );`, and insert the following line under it:
```
define('JWT_AUTH_SECRET_KEY', 'your-top-secret-key');
```
Afterwards, we will need to replace the part that says `your-top-secret-key` with a secure hash. We can do this by generating one using Wordpress Salt.

Visit [Wordpress Salt](https://api.wordpress.org/secret-key/1.1/salt/) and copy the string of characters between the single quotes. *(this is your secure hash)*

Screenshot: (string to copy is highlighted)
![Screenshot of Wordpress Salt with String to Copy being highlighted](https://lh3.googleusercontent.com/GjakZSjMbdEdqHykFAAQBlbTVBI1DDASppkFCcnF0f5MHwxO_g6-Lf5EXIaMgztls7Pntp6Izrs) 

Don't worry about this being secure, the hash is generated every time you refresh this page.

After you have copied the string from Salt, replace the `your-top-secret-key` with it.


## How to generate a token
If you read the [plugin documentation](https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/) they give examples for Angular, but unfortunately, not React.

The important thing that we CAN glean from the plugin, however, are the routes that JWT Plugin creates in the WP REST API.

For token generation, we will use the `/jwt-auth/v1/token` route.

We will need to build a request using the Fetch API in our React Application to generate the token. (you should test this in [Postman](https://www.getpostman.com/downloads/) first to be sure your server was setup correctly.)

The essential data that must be sent to generate a token is the `Username` and `Password`.

A basic request would look something like this:

```javascript
fetch(apiServer+"/jwt-auth/v1/token", { 
	method: "POST",
	headers: {
		'Content-Type': 'application/json', 
	},

	body: JSON.stringify({
		username: username,
		password: password
	})
})

	.then(response => {
		if(response.status !==  200){
			console.error('Connection error, code  ',response.status);
			return;
		}

		response.json(data => {
			// Your code here
		})
	})

	.catch(err=>{
		console.error(err)
	});
```

***Note:*** *apiServer is a variable you should define to contain the base host for all your Wordpress API requests. username and password are also variables that should be pulled from your login form. We will discuss the process of creating a login more later, so for the time being, you are welcome to hard code these values for testing.*

As you can see, we are making a POST request to the `/jwt-auth/v1/token` route with our `username` and `password` being serialized in the body using `JSON.stringify`. The rest of the request is your basic fetch with error catching. *Make sure to do something with the token once you have generated it.*

The return of this will be a Token, User Nice Name,  Email, Username

## How to validate a token


## Requests on a protected route



# Login Flow


## How to create a login


## Saving your Token



# Returning Visitors


## Checking for a token in LocalStorage

