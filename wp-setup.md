# Setting up your Python 3 Backend for JWT

**Technologies:** Wordpress, JWT plugin<br>

_____

When developing headless Wordpress websites with React, we will often times need to set up Users for our site. This means private routes/requests, and the need to authenticate said requests.

In the following sections, we will discuss some caveats and scenarios that will help you to better understand the workflow of setting up JWT on Wordpress.

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

![Screenshot of Wordpress Salt with String to Copy being highlighted](https://i.ibb.co/gg0CD6J/salt.png) 

Don't worry about this being secure, the hash is generated every time you refresh this page.

After you have copied the string from Salt, replace the `your-top-secret-key` with it.
