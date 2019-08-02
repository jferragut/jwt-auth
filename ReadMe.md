---


---

<h1 id="user-authentication-with-jwt">User Authentication with JWT</h1>
<p>by Jonathan J. Ferragut</p>
<p><strong>Technologies:</strong> Wordpress, JWT, Javascript, ReactJS</p>
<hr>
<p>When developing headless Wordpress websites with React, we will often times need to set up Users for our site. This means private routes/requests, and the need to authenticate said requests.</p>
<p>In the following sections, we will discuss some caveats and scenarios that will help you to better understand the workflow of setting up JWT on Wordpress and subsequently dealing with requests in React using Fetch API.</p>
<p>This document assumes that you have:</p>
<ol>
<li>Installed and configured a Wordpress install already</li>
<li>Have a Frontend developed in ReactJS</li>
<li>Are familiar with HTTP Requests</li>
</ol>
<p><strong>Note:</strong> All routes on the Wordpress API will start with <code>https://yourdomain.com/wp-json/</code>, where <code>yourdomain.com</code> should be replaced with the domain/host that you are using for your server. If you are using Gitpod, this will be a workspace URL that is assigned randomly to your instance. This will look something like:  <code>https://ffe4c36b-7ff9-4057-ba10-634bd14f8182.ws-us0.gitpod.io</code></p>
<h1 id="jwt-plugin">JWT Plugin</h1>
<p>The journey begins with the setup and install of JWT on your Wordpress backend. We’ll discuss a basic installation procedure, but if you require more, you can defer to the plugin documentation.</p>
<p>Please make sure to Install the JWT Authentication plugin which can be found here: <a href="https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/">JWT Plugin Download Page</a></p>
<h2 id="how-to-setup-jwt-on-wordpress">How to setup JWT on Wordpress</h2>
<p>Setting up the JWT plugin starts with modifying the <code>.htaccess</code> file, which controls settings for the apache server. After that, you will need to add a configuration value to the <code>wp-config.php</code> file.</p>
<p><strong>Step 1:</strong><br>
Open the <code>.htaccess</code> file and look for the line that says <code>RewriteEngine On</code>.  Immediately after this line, insert the following two lines of code:</p>
<pre><code>RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]
</code></pre>
<p>This will enable authorization on routes and require that you use Bearer Tokens access secure routes. <em>Please make sure to save your changes.</em></p>
<p><em><strong>Note:</strong></em> <em>If you encounter a 503 error with your fetch on the frontend , check to see if the settings in your <code>.htaccess</code> file have been overwritten.</em></p>
<p><strong>Step 2:</strong><br>
Open the <code>wp-config.php</code> file and search for a line that says <code>define( 'WP_DEBUG', true );</code>, and insert the following line under it:</p>
<pre><code>define('JWT_AUTH_SECRET_KEY', 'your-top-secret-key');
</code></pre>
<p>Afterwards, we will need to replace the part that says <code>your-top-secret-key</code> with a secure hash. We can do this by generating one using Wordpress Salt.</p>
<p>Visit <a href="https://api.wordpress.org/secret-key/1.1/salt/">Wordpress Salt</a> and copy the string of characters between the single quotes. <em>(this is your secure hash)</em></p>
<p>Screenshot: (string to copy is highlighted)<br>
<img src="https://lh3.googleusercontent.com/GjakZSjMbdEdqHykFAAQBlbTVBI1DDASppkFCcnF0f5MHwxO_g6-Lf5EXIaMgztls7Pntp6Izrs" alt="Screenshot of Wordpress Salt with String to Copy being highlighted"><br>
Don’t worry about this being secure, the hash is generated every time you refresh this page.</p>
<p>After you have copied the string from Salt, replace the <code>your-top-secret-key</code> with it.</p>
<h2 id="how-to-generate-a-token">How to generate a token</h2>
<p>If you read the <a href="https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/">plugin documentation</a> they give examples for Angular, but unfortunately, not React.</p>
<p>The important thing that we CAN glean from the plugin, however, are the routes that JWT Plugin creates in the WP REST API.</p>
<p>For token generation, we will use the <code>/jwt-auth/v1/token</code> route.</p>
<p>We will need to build a request using the Fetch API in our React Application to generate the token. (you should test this in <a href="https://www.getpostman.com/downloads/">Postman</a> first to be sure your server was setup correctly.)</p>
<p>The essential data that must be sent to generate a token is the <code>Username</code> and <code>Password</code>.</p>
<p>A basic request would look something like this:</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token function">fetch</span><span class="token punctuation">(</span>apiServer<span class="token operator">+</span><span class="token string">"/jwt-auth/v1/token"</span><span class="token punctuation">,</span> <span class="token punctuation">{</span> 
	method<span class="token punctuation">:</span> <span class="token string">"POST"</span><span class="token punctuation">,</span>
	headers<span class="token punctuation">:</span> <span class="token punctuation">{</span>
		<span class="token string">'Content-Type'</span><span class="token punctuation">:</span> <span class="token string">'application/json'</span><span class="token punctuation">,</span> 
	<span class="token punctuation">}</span><span class="token punctuation">,</span>

	body<span class="token punctuation">:</span> JSON<span class="token punctuation">.</span><span class="token function">stringify</span><span class="token punctuation">(</span><span class="token punctuation">{</span>
		username<span class="token punctuation">:</span> username<span class="token punctuation">,</span>
		password<span class="token punctuation">:</span> password
	<span class="token punctuation">}</span><span class="token punctuation">)</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span>

	<span class="token punctuation">.</span><span class="token function">then</span><span class="token punctuation">(</span>response <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
		<span class="token keyword">if</span><span class="token punctuation">(</span>response<span class="token punctuation">.</span>status <span class="token operator">!==</span>  <span class="token number">200</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
			console<span class="token punctuation">.</span><span class="token function">error</span><span class="token punctuation">(</span><span class="token string">'Connection error, code  '</span><span class="token punctuation">,</span>response<span class="token punctuation">.</span>status<span class="token punctuation">)</span><span class="token punctuation">;</span>
			<span class="token keyword">return</span><span class="token punctuation">;</span>
		<span class="token punctuation">}</span>

		response<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span>data <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
			<span class="token comment">// Your code here</span>
		<span class="token punctuation">}</span><span class="token punctuation">)</span>
	<span class="token punctuation">}</span><span class="token punctuation">)</span>

	<span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>err<span class="token operator">=&gt;</span><span class="token punctuation">{</span>
		console<span class="token punctuation">.</span><span class="token function">error</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span>
	<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p><em><strong>Note:</strong></em> <em>apiServer is a variable you should define to contain the base host for all your Wordpress API requests. username and password are also variables that should be pulled from your login form. We will discuss the process of creating a login more later, so for the time being, you are welcome to hard code these values for testing.</em></p>
<p>As you can see, we are making a POST request to the <code>/jwt-auth/v1/token</code> route with our <code>username</code> and <code>password</code> being serialized in the body using <code>JSON.stringify</code>. The rest of the request is your basic fetch with error catching. <em>Make sure to do something with the token once you have generated it.</em></p>
<p>The return of this will be a Token, User Nice Name,  Email, Username</p>
<h2 id="how-to-validate-a-token">How to validate a token</h2>
<h2 id="requests-on-a-protected-route">Requests on a protected route</h2>
<h1 id="login-flow">Login Flow</h1>
<h2 id="how-to-create-a-login">How to create a login</h2>
<h2 id="saving-your-token">Saving your Token</h2>
<h1 id="returning-visitors">Returning Visitors</h1>
<h2 id="checking-for-a-token-in-localstorage">Checking for a token in LocalStorage</h2>

