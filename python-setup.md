# Setting up your Python 3 Backend for JWT

**Boilerplate *(optional)*:** [Python 3 with Flask, pipenv, and sqlAlchemy](https://github.com/4GeeksAcademy/flask-rest-hello)

**Technologies:** Python 3, pipenv, flask,  flask-jwt-simple

**Documentation** 
Flask-jwt-simple:   [Click for Documentation](https://flask-jwt-simple.readthedocs.io/en/latest/index.html)

_____

## Setting up the boilerplate

## Installing Flask-JWT-Simple

Assuming you are using the boilerplate noted above, you can install using `pipenv`:

- `pipenv install flask-jwt-simple`

Otherwise, you would use `pip`:

- `pip install flask-jwt-simple`

## Working with JWT and Protected Routes

Before you get started, you need to import the package to your project. This will occur on your main file where routes will be declared. If you are using the 4Geeks Boilerplate mentioned above, you will need to add it to `main.py` which is located in the `src` folder.

The import will look something like this:

```python
from flask_jwt_simple import (
    JWTManager, jwt_required, create_jwt, get_jwt_identity
)
```

All this does is call in the basic functionality from the flask-jwt-simple package. Next, after the `app=` declaration line, we will need to initialize jwt.

```python
app.config['JWT_SECRET_KEY'] = 'super-secret'  # Change this!
jwt = JWTManager(app)
```

On the first line above, you need to replace the `super-secret` value with your JWT Secret key. To generate a key you can utilize any of a number of online resources.

I would recommend you using a sha2 key generator, which will help you create a secure key. Here is an example:
[Hash Calculator](https://xorbin.com/tools/sha256-hash-calculator)

![Screenshot of Hash Generator](https://ibb.co/KmyRpP5)

In the screenshot, you can see that for this specific site, I simply copy and pasted a phrase (in this case a quote from Dune) and clicked the `Calculate SHA256 Hash` button.

That takes care of the initial import and setup for your app.

## Creating a Login Route

In the Flask-JWT-Simple docs, they have a simple login route that you can pretty much copy but we should understand what is going on.

```python
# Provide a method to create access tokens. The create_jwt()
# function is used to actually generate the token
@app.route('/login', methods=['POST'])
def login():
    if not request.is_json:
        return jsonify({"msg": "Missing JSON in request"}), 400

    params = request.get_json()
    username = params.get('username', None)
    password = params.get('password', None)

    if not username:
        return jsonify({"msg": "Missing username parameter"}), 400
    if not password:
        return jsonify({"msg": "Missing password parameter"}), 400

    if username != 'test' or password != 'test':
        return jsonify({"msg": "Bad username or password"}), 401

    # Identity can be any data that is json serializable
    ret = {'jwt': create_jwt(identity=username)}
    return jsonify(ret), 200
```

Firstly, the route will be available on `/login` and accepts a `POST` request. After copying this, you can test it in Postman to make sure it is working properly.

Our initial `IF` statement is testing to see if the response is a JSON and will return a 400 if you don't have a JSON in the request. It's important to note on the next few lines we see that our JSON should have the properties `username` and `password`.

Next, the code will test for the presence of both requeired properties and return a 400 if either is missing. 

There is an `IF` statement that is checking to make sure the user and password are set to `test` for the sake of this example. In your actual code, you will want to remove these lines and instead insert a query using your ORM of choice (or SQL directly) to check your database and verify the user credentials match.

Finally, on the line that reads `ret = {'jwt': create_jwt(identity=username)}` we are setting up to retrun a json with a token value stored in the `jwt` property.

## Protected Routes

Now that you have a way of logging a user in and returning a token, you need to be able to set up which of your Flask routes are protected. We do this using decorators.

All you have to do is take any route and add the `@jwt_required` decorator to it. Here is an example:

```python
# Protect a view with jwt_required, which requires a valid jwt
# to be present in the headers.
@app.route('/protected', methods=['GET'])
@jwt_required
def protected():
    # Access the identity of the current user with get_jwt_identity
    return jsonify({'hello_from': get_jwt_identity()}), 200
```

If you look at the above code, you can see that on the line after your `@app.route` declaration, you have a `@jwt_required` decorator. This is all that is needed to tell your app that the current route needs a token to access it.

All you need to access it from the frontend is to send the token as a Bearer token in your request.

This is the basis for all tokenization using JWT. For other advanced details about the library, take a look at the documentation that is linked at the top of this markdown file.
