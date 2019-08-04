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

<br>

## JWT Plugin

The journey begins with the setup and install of JWT on your Wordpress backend. We'll discuss a basic installation procedure, but if you require more, you can defer to the plugin documentation.

Please make sure to Install the JWT Authentication plugin which can be found here: [JWT Plugin Download Page](https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/)

<br>

### How to setup JWT on Wordpress

Setting up the JWT plugin starts with modifying the `.htaccess` file, which controls settings for the apache server. After that, you will need to add a configuration value to the `wp-config.php` file.

**Step 1:** 
Open the `.htaccess` file and look for the line that says `RewriteEngine On`.  Immediately after this line, insert the following two lines of code:

```none
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]
```

This will enable authorization on routes and require that you use Bearer Tokens access secure routes. *Please make sure to save your changes.*

***Note:*** *If you encounter a 503 error with your fetch on the frontend , check to see if the settings in your `.htaccess` file have been overwritten.*

**Step 2:**  
Open the `wp-config.php` file and search for a line that says `define( 'WP_DEBUG', true );`, and insert the following line under it:

```php
define('JWT_AUTH_SECRET_KEY', 'your-top-secret-key');
```

Afterwards, we will need to replace the part that says `your-top-secret-key` with a secure hash. We can do this by generating one using Wordpress Salt.

Visit [Wordpress Salt](https://api.wordpress.org/secret-key/1.1/salt/) and copy the string of characters between the single quotes. *(this is your secure hash)*

Screenshot: (string to copy is highlighted)

![Screenshot of Wordpress Salt with String to Copy being highlighted](https://lh3.googleusercontent.com/GjakZSjMbdEdqHykFAAQBlbTVBI1DDASppkFCcnF0f5MHwxO_g6-Lf5EXIaMgztls7Pntp6Izrs) 

Don't worry about this being secure, the hash is generated every time you refresh this page.

After you have copied the string from Salt, replace the `your-top-secret-key` with it.

<br>

### How to generate a token

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

The return of this will include the following fields: Token, User Nicename,  Email, and User Display Name.

**Example:**

```json
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9qd3QuZGV2IiwiaWF0IjoxNDM4NTcxMDUwLCJuYmYiOjE0Mzg1NzEwNTAsImV4cCI6MTQzOTE3NTg1MCwiZGF0YSI6eyJ1c2VyIjp7ImlkIjoiMSJ9fX0.YNe6AyWW4B7ZwfFE5wJ0O6qQ8QFcYizimDmBy6hCH_8",
    "user_display_name": "admin",
    "user_email": "admin@localhost.dev",
    "user_nicename": "admin"
}
```

<br>

### How to validate a token

As we did with the creation of token, you can also validate the token with a predefined route.

JWT Plugin provides the following route: `/jwt-auth/v1/token/validate`

Let's look at a slimmed down fetch that will validate your token. This example will assume the same apiServer variable exists, also, we use a new `token` variable that should hold your token.

```javascript
fetch(apiServer+"/jwt-auth/v1/token", { 
	method: "POST",
	headers: {
		'Content-Type': 'application/json', 
		Authorization: 'Bearer ' + token
	},
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

If your token is valid, the response should look something like this:

```json
{
  "code": "jwt_auth_valid_token",
  "data": {
    "status": 200
  }
}
```

<br>

### Requests on a protected route

When making requests to a protected route, you will need to append the Authorization header with the token.

The fetch component would look something like:

```javascript
fetch(apiServer+"/myRoute", {  
    method: "POST",  
    headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + token
    },
    body: JSON.stringify(payload)  
})
```

Where the `method` would vary depending on your endpoint, and the payload would be anything that you need to serialize in the body of your request.

<br>

## Login Flow

User login will consist of a few of cases. In the following sections, we will cover each case so that you can better understand setting this up in your application.

<br>

### New User Registration

First, when someone is not a current user on the site, you will need to allow them to register.

This is done by creating a registration form view in your application. The form should be stateful and should then submit via a fetch `POST` request that will send the state of the component to the Wordpress User Creation API.

***Note:*** If you are building this using the 4 Geeks Academy boilerplate, you will be creating an action to `createUser()` in your context store. 

The Wordpress Create User API is located at: `https://[yourdomain.com]/wp-json/wp/v2/users`

If you look at the flowchart below, you will see an example of how the Registration should go. Note that if the user fails, we should throw error(s) and allow the user to fix them on the registration form.


![New User Registration Flowchart](https://lh3.googleusercontent.com/MHHdToT2A7Vf6z9Xtixk0EG_ID6xCji_aPe8Wll72BB3W-wAP2kxhodSQK4NKoiQ48X6oyMx4VUo3261LEhAyzjbgoQrFduAnbY7hlfJnhOcK1ZJ-709oBr-pz9ofGHeF3Eil_t7wEe43l477Wkces71faVUzpqhwlecu8NFXvoKnZynCd2djr-fhmDf9ldjPaUlXbm67TD8lRqxNV9Ec632padm9B-p6byp6JgkCBxZ0gEaflZWcn6a-eASAsuHixpP7KxIFInPxJIw8MpDVir0hMqYatNnFgcSrhvRFA8rEHsy6-UIv6Gv6c89clw7Rw-iLAYPSbQWKz5NA0HLoZOEHUCtAihvSU6Q6tdbD29czgbMUQialLXs5cOhre_oM1c5-chEIFF0c4nTV3dvlVEcLYxXgm3ZW4sz73nE5hp7t_QQ5stIMyq8yig8zGBKJwl2twE1uV-kRtPGAdzaNpJdqIBRu-NVUGeqG9W1jB_zDbteWxNkggkn5w461Gm_ReH1UlJdqV01yAak8j530j_h1LeDsSgneyoYt4C0m6kzapRg6C_3irZnHRc40viGmZ-1oE-YWb_FJ3Y1ySvRQmq39R8HSazD9pI9Ueh9ege2yHW6VbS2cfb11wv5sQYyN3cpg4V0ENUu3CQfA0YMXPzSNph80g=w471-h421-no)

<br>

An example of the action that you should create to create a user is as follows:

```javascript
fetch(apiServer+"/wp/v2/users", {
    method: "POST",
    headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer '+token
    },
    body: JSON.stringify(userData)
})
```

Just like in our previous examples, apiServer should be your wordpress server location. The userData that we are stringifying in the body of the request should contain at minimum, the following required elements:

```json
{
	"username": "username",
	"email": "email",
	"password": "password",
}
```

Additional information is allowed and can be viewed here: [Wordpress User Route Arguments](https://developer.wordpress.org/rest-api/reference/users/#arguments-2)

Since JWT is setup for protected routes, we will need to send a token when we connect to the endpoint.

The best way that you can do this is to create a user account for the API and it's requests. Then you can create a method that generates a token for that user, so that it can be attached to any requests on the endpoint.

***Note:*** I would also recommend storing this admin token in your application store so that you can check if there is already a token available when you perform requests.


<br>

### How to create a login


<br>

### Saving your Token


<br>

## Returning Visitors

<br>

### Checking for a token in LocalStorage

