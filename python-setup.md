# Setting up your Python 3 Backend for JWT

**Boilerplate *(optional)*:** [Python 3 with Flask, pipenv, and sqlAlchemy](https://github.com/4GeeksAcademy/flask-rest-hello)

**Technologies:** Python 3, pipenv, flask,  flask-jwt-simple

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
