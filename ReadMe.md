# User Authentication with JWT
by Jonathan J. Ferragut

**Technologies:** Wordpress, JWT, Javascript, ReactJS, React Context API, Fetch API <br>
**Boilerplate *(optional)*:** [React WebApp with Context](https://github.com/4GeeksAcademy/react-hello-webapp)
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

		response.json().then().then(data => {
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

		response.json().then(data => {
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


![New User Registration Flowchart](https://lh3.googleusercontent.com/e1YxloTT1R3grj6f2RjkvmqGdu4SJmpJgxzTzQ-DINJtSXIYOKWOIic-xqL3fGqHY7x0Mz55qpUsQnQYSHawX5UfXHKswPEF3eDdhR5w5o1jxfnKshFwO8NPxjHZscLEizSmli2f=w2400)

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

### How to create a user login

Whenever you are creating your login, it's good to start by understanding the flow of your component.

![Login Flow Diagram](https://lh3.googleusercontent.com/7SkoAFUY4z-LEvWOq7_7LQ2_zLeL6-Fj8kCUM8aGYhtpCCCjIn06E3KZjVlBouyXNiK5SG4efIAZoFQWhvHkiAg5QWk26wIIBggNGGh4dZ08vHtLUTma0wt2EiwNllZcG9r_TCJZ=w2400)

Your login form should be a stateful component that tracks the username and password of anyone trying to login. (I also recommend using an input type password for your password field to obfuscate the password to potential onlookers.)

At this point in your app, you will create an action in your store that generates a token for a user when provided a `username` and `password` as input.

```javascript
//example of action when using the 4Geeks context boilerplate
generateToken: (username,password)=>{
	let store = getStore();
	// lets assume that you have a variable in your store for user to store user data when logged in
	// and another variable for the login status

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
			if(response.status !== 200){
				return 'Connection error: '+response.status;
			}

			response.json().then(data => {
				store.user = data;
				store.loggedIn = true;
				setStore({ store });
			})
		})
		.catch(err=>{
			return err;
		});
}
```

This function should return an error if there is one and if not, will set `loggedIn` to true. Your component should be reactive to this and then respond in each case.

If user `loggedIn` is true, store the current token in the user's localStorage. Otherwise, display error context and allow the user to react accordingly.

<br>

### Saving your Token

Now that you have obtained the token, the next step is to save it in localStorage for the user. This is super easy to do:

```javascript
localStorage.setItem('yourApp-userData', JSON.stringify({
	token: token,
	user_display_name: user_display_name,
	user_email: user_email,
	user_nicename: user_nicename
})
);
```
We stringify the data to avoid any issues with the object when storing it. If we didn't stringify, we would get a return of `[Object,Object]` when we try to access the data.

Now, if you already have this data in your store and are within a context consumer when you set the token (as we mentioned previously), you can do the following:

```javascript
localStorage.setItem('yourApp-userData', JSON.stringify(store.user));
```

*(assuming that your store has been passed in as `store` and that your user token data was stored in `store.user`)*

Of course, during your current session, you don't need to access any of the data from the localStorage because you have it in your store already. So whenever you check if a user is loggedIn, simply check the flag in your store and then make sure to pull data from the store.

<br>

## Returning Visitors

When visitors are returning to your site we need to check if they have a token to know if they are still valid or need to log in again. The cool thing is that we can simply check all users to see if there is anything from us in localStorage and if not, we can assume they are a new user.

If there is a token in localStorage, we want to validate it.

![Validating Returning Users Flowchart](https://lh3.googleusercontent.com/_g6pb8mo4oVYo9qBqBPOjmVC8j5qQd1AwRbYYw-whI_rAy_ilcbaEQPZqOywlZpBojIv5x3S7jrcQGdNJ1NNu9fdToVX07ag3V26W79n8DhM0Rcjfgy6NBHEhvibJRxhgrcoQDCW=w2400)


<br>

### Checking for a token in LocalStorage

The above sequence is a bit more involved with more forks in it. Initially, as mentioned, we check all users that access the app to see if they have a Token in localStorage. We do so similarly to how we set the token.

```javascript
let tokenCheck = JSON.parse(localStorage.getItem)('yourApp-userData');

if(tokenCheck!==null){
	// token is present
}else{
	// token is not present
}
```

In the section above, we check to see if the token exists in localStorage. That function will return `null` if there is no data in localStorage at the specified key. 

This allows us to perform a conditional check and act accordingly. In the first area that states `token is present`, we would want to then validate the token. If it is valid, we would want to then store the token in the store so that we can access the data in our views.

***Note:*** *You'll want to make sure that you also toggle the `loggedIn` to true on your store if you have validated and stored the user data in your app.*

Else, if the `token is not present`, we want to then maybe grab the `user_nicename` to display on a welcome in the navbar, but don't force the user to log in until they attempt to visit a protected route.

*(Think about how stores like Amazon greet users. They welcome you in the top bar with your first name, but they don't actually tell you to log in all the time. In actuality, they are only requesting login if your token has expired or if you are on a device they don't recognize.)*

Looking at our last code snippet again, let's assume you are running that from an action in your store.

```javascript
checkToken: ()=>{
	let store = getStore();
	let tokenCheck = JSON.parse(localStorage.getItem('yourApp-userData'));

	if(tokenCheck!==null){
		// set current user data to store
		store.user=tokenCheck;
		setStore({store});

		// fetch to validate current token
		fetch(apiServer+"/jwt-auth/v1/token/validate", {
			method: "POST",
			headers: {
				'Content-Type': 'application/json',
				Authorization: 'Bearer '+token
			}
		})
			.then(response => {
				response.json().then(data => {
					// token is valid
					setStore({
						...store,
						loggedIn: true
					});
				})
			})
			.catch(err=>{
				return err;
			});
	}else{
		setStore({
			...store,
			loggedIn: false,
			user: null
		});
	}
}
```

The above function will check if there is anything in localStorage. If there is , we set current data in the store and then do a validate on the token. When data comes back, if it's valid, we set `loggedIn` to true in the store, otherwise we don't.

Finally, if there is nothing in the localStorage, we set the `user` to null in the store and set `loggedIn` to false.

<br>

______

## Conclusion

This write-up was meant as a top level overview of users and tokens with your application and not as an in-depth breakdown. You will probably need to tailor the approach to your specific app, but hopefully you have gained some insight into working with JWT and tokenization.

If you have any questions, feel free to post an issue on this repository or to ask your instructor.
