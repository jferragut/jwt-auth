# Considerations for setting up tokenization in the FrontEnd using ReactJS

**Boilerplate *(optional)*:** [React WebApp with Context](https://github.com/4GeeksAcademy/react-hello-webapp)

## How to generate a token

To generate a token, you typically need to send a request to your token route which will in essence, log the user in with your API.

We will need to build a request using the Fetch API in our React Application to generate the token. (you should test this in [Postman](https://www.getpostman.com/downloads/) first to be sure your server was setup correctly.)

The essential data that must be sent to generate a token is the `Username` and `Password`.

A basic request would look something like this:

```javascript
fetch(apiHost+"/jwt-auth/v1/token", {
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
		if(!response.ok){
			throw Error(response.statusText);
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

***Note:*** *apiHost is a variable you should define to contain the base host for all your  API requests. username and password are also variables that should be pulled from your login form. We will discuss the process of creating a login more later, so for the time being, you are welcome to hard code these values for testing. It's also important to note that the route provided after apiHost is unique to your api. This means it will be different depending on your backend setup.*

As you can see, we are making a POST request to the `/jwt-auth/v1/token` (if you are using wordpress. otherwise, this should reflect the route for your python api) route with our `username` and `password` being serialized in the body using `JSON.stringify`. The rest of the request is your basic fetch with error catching. *Make sure to do something with the token once you have generated it.*

By default, the return of this will include the following fields:
- On Wordpress -  Token, User Nicename,  Email, and User Display Name.
- On Python/Flask - jwt (contains your token)

**Example:**

For wordpress:
```json
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9qd3QuZGV2IiwiaWF0IjoxNDM4NTcxMDUwLCJuYmYiOjE0Mzg1NzEwNTAsImV4cCI6MTQzOTE3NTg1MCwiZGF0YSI6eyJ1c2VyIjp7ImlkIjoiMSJ9fX0.YNe6AyWW4B7ZwfFE5wJ0O6qQ8QFcYizimDmBy6hCH_8",
    "user_display_name": "admin",
    "user_email": "admin@localhost.dev",
    "user_nicename": "admin"
}
```

For python/flask:
```json
{
    "jwt": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1MDM1OTk3MTgsImlhdCI6MTUwMzU5NjExOCwibmJmIjoxNTAzNTk2MTE4LCJzdWIiOiJ0ZXN0In0.G2GnN9NgvvmSKgRDGok0OjAyDWkG_qCn4FTxSfPUXDY"
}
```

### How to validate a token

As we did with the creation of token, you can also validate the token with a predefined route. This method is built-in to the wordpress plugin version of JWT, but not a default in the basic setup of the python version, so if you want a validation route, you will have to create on and add to your API.

JWT Plugin provides the following route: `/jwt-auth/v1/token/validate`

Let's look at a slimmed down fetch that will validate your token. This example will assume the same apiHost variable exists, also, we use a new `token` variable that should hold your token.

```javascript
fetch(apiHost+"/jwt-auth/v1/token", { 
	method: "POST",
	headers: {
		'Content-Type': 'application/json', 
		Authorization: 'Bearer ' + token
	},
})

	.then(response => {
		if(!response.ok){
			throw Error(response.statusText);
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
fetch(apiHost+"/myRoute", {  
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

This is done by creating a registration form view in your application. The form should be stateful and should then submit via a fetch `POST` request that will send the state of the component to your api endpoint.

In wordpress, this will mean utilizing the Wordpress User Creation API.

The Wordpress Create User API is located at: `https://[yourdomain.com]/wp-json/wp/v2/users`

***Note:*** If you are building this using the 4 Geeks Academy boilerplate, you will be creating an action to `createUser()` in your context store. 

If you look at the flowchart below, you will see an example of how the Registration should go. Note that if the user fails, we should throw error(s) and allow the user to fix them on the registration form.


![New User Registration Flowchart](https://i.ibb.co/w6HLLdb/Registration.png)

<br>

An example of the action that you should create to create a user is as follows:

```javascript
fetch(apiHost+"/wp/v2/users", {
    method: "POST",
    headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer '+token
    },
    body: JSON.stringify(userData)
})
```

Just like in our previous examples, apiHost should be your wordpress server location. Also, make sure that if you are not using Wordpress, you update the endpoint to reflect your create user endpoint.

For wordpress, the userData that we are stringifying in the body of the request should contain at minimum, the following required elements:

```json
{
	"username": "username",
	"email": "email",
	"password": "password",
}
```

Additional information is allowed and can be viewed here: [Wordpress User Route Arguments](https://developer.wordpress.org/rest-api/reference/users/#arguments-2)

Since JWT is setup for protected routes, we will need to send a token when we connect to the endpoint.

For Wordpress, The best way that you can do this is to create a user account for the API and it's requests. Then you can create a method that generates a token for that user, so that it can be attached to any requests on the endpoint.

***Note:*** I would also recommend storing this admin token in your application store so that you can check if there is already a token available when you perform requests.

<br>

### How to create a user login

Whenever you are creating your login, it's good to start by understanding the flow of your component.

![Login Flow Diagram](https://i.ibb.co/KF6zQ2p/Login-Flow.png)

Your login form should be a stateful component that tracks the username and password of anyone trying to login. (I also recommend using an input type password for your password field to obfuscate the password to potential onlookers.)

At this point in your app, you will create an action in your store that generates a token for a user when provided a `username` and `password` as input.

```javascript
//example of action when using the 4Geeks context boilerplate
generateToken: (username,password)=>{
	// lets assume that you have a variable in your store for user to store user data when logged in
	// and another variable for the login status

	fetch(apiHost+"/jwt-auth/v1/token", {
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
			if(!response.ok){
				throw Error(response.statusText);
			}

			response.json().then(data => {
				// if you have a different store setup, replace the setStore() with your own method of updating your store info.
				setStore({
					user: data,
					loggedIn: true
				});
			})
		})
		.catch(err=>{
			console.error(err);
		});
}
```

This function should log an error if there is one and if not, will set `loggedIn` to true. Your component should be reactive to this and then respond in each case.

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

Again, if you are using Python, your information being stored may look different. That is ok. At minimum, we need to store the username and the token, but having a nicename really goes a long way towards being friendly with your users.

Now, if you already have this data in your store and are within a context consumer when you set the token (as we mentioned previously), you can do the following:

```javascript
localStorage.setItem('yourApp-userData', JSON.stringify(store.user));
```

*(assuming that your store has been passed in as `store` and that your user token data was stored in `store.user`)*

Of course, during your current session, you don't need to access any of the data from the localStorage because you have it in your store already. So whenever you check if a user is loggedIn, simply check the flag in your store and then make sure to pull data from the store.

The only time you should be checking for the token is if the user is not flagged as loggedIn on your app.

<br>

## Returning Visitors

When visitors are returning to your site we need to check if they have a token to know if they are still valid or need to log in again. The cool thing is that we can simply check all users to see if there is anything from us in localStorage and if not, we can assume they are a new user.

If there is a token in localStorage, we want to validate it.

![Validating Returning Users Flowchart](https://i.ibb.co/QY4ZW7X/Check-for-Token.png)

<br>

### Checking for a token in LocalStorage

The above sequence is a bit more involved with more forks in it. Initially, as mentioned, we check all users that access the app to see if they have a Token in localStorage. We do so similarly to how we set the token.

```javascript
let tokenCheck = JSON.parse(localStorage.getItem)('yourApp-userData');

if(tokenCheck!==null){
	// token is present, so do something (set loggedIn, maybe?)
}else{
	// token is not present, redirect to login flow
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
	let tokenCheck = JSON.parse(localStorage.getItem('yourApp-userData'));

	if(tokenCheck!==null){
		// set current user data to store
		setStore({ user: tokenCheck });

		// fetch to validate current token
		fetch(apiHost+"/jwt-auth/v1/token/validate", {
			method: "POST",
			headers: {
				'Content-Type': 'application/json',
				Authorization: 'Bearer '+token
			}
		})
			.then(response => {
				response.json().then(data => {
					// token is valid
					setStore({ loggedIn: true });
				})
			})
			.catch(err=>{
				return err;
			});
	}else{
		setStore({
			loggedIn: false,
			user: null
		});
	}
}
```

The above function will check if there is anything in localStorage. If there is , we set current data in the store and then do a validate on the token. When data comes back, if it's valid, we set `loggedIn` to true in the store, otherwise we don't.

Finally, if there is nothing in the localStorage, we set the `user` to null in the store and set `loggedIn` to false.
