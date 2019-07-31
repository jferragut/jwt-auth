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
<h1 id="jwt-plugin">JWT Plugin</h1>
<p>The journey begins with the setup and install of JWT on your Wordpress backend. We’ll discuss a basic installation procedure, but if you require more, you can defer to the plugin documentation.</p>
<h2 id="how-to-setup-jwt-on-wordpress">How to setup JWT on Wordpress</h2>
<h2 id="how-to-generate-a-token">How to generate a token</h2>
<h2 id="how-to-validate-a-token">How to validate a token</h2>
<h2 id="requests-on-a-protected-route">Requests on a protected route</h2>
<h1 id="login-flow">Login Flow</h1>
<h2 id="how-to-create-a-login">How to create a login</h2>
<h2 id="saving-your-token">Saving your Token</h2>
<h1 id="returning-visitors">Returning Visitors</h1>
<h2 id="checking-for-a-token-in-localstorage">Checking for a token in LocalStorage</h2>

