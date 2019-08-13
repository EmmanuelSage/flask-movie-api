# flask-movie-api
This is a movie api to learn flask

Steps

> Setup environment

- `$ python3 -m venv env` create env

- `$ source env/bin/activate ` activate env

- `$ pip install flask flask-sqlalchemy` install flask and flask-sqlalchemy

- `$ pip install --upgrade pip` upgrade pip if need be for venv

- create api folder

- `$ export FLASK_APP=api` : this allows us to use flask run on whats in the api folder

- `$ export FLASK_DEBUG=1`: this is so when we run the app it would be in debug mode


>  create an __init__.py and add this to create app and set config

```py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)

    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'

    return app

```

- we imported flask and flask_sqlalchemy,
- instantiated a new db instance
- created a create_app method in which we initialized our flask app
- and set the database to sqlite, :/// means relative path and ://// means absolute path to where we want the db file
- we then return app so it can be used

> create a views.py and add this

```py
from flask import Blueprint, jsonify,

main = Blueprint('main', __name__)

@main.route('/movie', methods=['POST'])
def add_movie():

    return 'Done', 201

@main.route('/movies')
def movies():

    movies = []

    return jsonify({'movies' : movies})
```

- here we import blueprint(this iis used to separate our api in flask) and jsonify(to return an array through a json object)

- we initiaize the main variable with blueprint

- we define a POST route to return 'Done' and 201

- we define a get route to get all movies, here we define a movies array and return the jsonified object

> Import the views into the \__init\__.py file

```py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)

    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'

    from .views import main
    app.register_blueprint(main)

    return app
```

- we import the views in the method because that is where the flask app is defined else it would not work, this would allow all the imports to be in the correct order as we are separating with blueprint

- on the app object we register the blueprint called main

- run `$ flask run` to run app

- test in postman and you should be able to get the routes

> INitialize a db object with SQLAlchemy and initialize with init_app in create_app
- normally we would initailize app in SQLAlchemy, but since we are using the create_app method we would use init_app inside create_app

```py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)

    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'

    db.init_app(app)

    from .views import main
    app.register_blueprint(main)

    return app
```

> create a models.py file and add this:
```py
from . import db

class Movie(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(50))
    rating = db.Column(db.Integer)
```
- here we import db from our project using the .
- we then extend the db.models class to create the Movie table
- we create the table columns id, title and rating using db.Column


> now to create the database follow the terminal commands

```bash
(env)$ python

>>> from api.models import Movie
>>> from api import db, create_app
>>> db.create_all(app=create_app())
```

- in your terminal type python to open the python environment
- we import Movie from api.models so everything is imported in the right order, but we don't actually use this
- we then import db and create_app
- we use db.create_all to create our tables from all the models in the project passing it our create_app

- you can check the sqlite database by typing
```bash
(env)$ sqlite3 api/database.db
sqlite> .tables
movie
sqlite> select * from movie;
sqlite> .exit
```

> add this to the views.py file

```py

from flask import Blueprint, jsonify, request
from . import db
from .models import Movie

main = Blueprint('main', __name__)

@main.route('/movie', methods=['POST'])
def add_movie():
    movie_data = request.get_json()

    new_movie = Movie(title=movie_data['title'], rating=movie_data['rating'])

    db.session.add(new_movie)
    db.session.commit()

    return 'Done', 201

@main.route('/movies')
def movies():
    movies = []

    return jsonify({'movies' : movies})
```

- we import request so we can get the request json data from the client
- we import db and Movie model, . means import from \__init\__.py and .models means import from models
- we get the json sent from request.get_json into the movie_data as a dictionary
- we now create a new movie object from the Movie model into new_movie
- we add to db as db.session.add(new_movie)
- and we commit it with db.commit()
- you can test this by sending a post request in postman to the url
- you can check the sqlite database using the command precviously shown to confirm


> to get all movies add this to the movie method in views.py

```py

from flask import Blueprint, jsonify, request
from . import db
from .models import Movie

main = Blueprint('main', __name__)

@main.route('/movie', methods=['POST'])
def add_movie():
    movie_data = request.get_json()

    new_movie = Movie(title=movie_data['title'], rating=movie_data['rating'])

    db.session.add(new_movie)
    db.session.commit()

    return 'Done', 201

@main.route('/movies')
def movies():
    movie_list = Movie.query.all()
    movies = []

    for movie in movie_list:
        movies.append({'title' : movie.title, 'rating' : movie.rating})

    return jsonify({'movies' : movies})
```

- we use `Movie.query.all()` to get all the movies
- we can't send movies_list directly as it is a sqlalchemy query object, we loop through it to create a dicitionary we can send




