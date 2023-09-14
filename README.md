# Creating RESTFull API with CRUD application using Django Rest Framework (DRF)
[Django REST framework](http://www.django-rest-framework.org/) is an extension of the Django web framework for Python, designed to simplify the creation and management of RESTful APIs. It equips developers with a set of powerful tools and features, including serializers for data conversion, class-based views for handling HTTP requests, and built-in support for authentication and permissions. DRF also streamlines tasks like pagination, request parsing, and response rendering, making it a popular choice for building robust and scalable APIs. With DRF, developers can efficiently create, secure, and interact with REST APIs while leveraging Django's web development capabilities.

**CRUD (Create, Read, Update, Delete)** represents the four fundamental operations in database applications. In the context of a RESTful API, "Create" involves adding new data using HTTP POST requests, "Read" retrieves data with HTTP GET requests, "Update" modifies existing data using HTTP PUT or PATCH requests, and "Delete" removes data through HTTP DELETE requests. These operations are essential for managing resources in a database and are seamlessly implemented in Django REST framework, making it a go-to choice for developers looking to create APIs that perform these actions reliably and efficiently while adhering to REST principles.

## Requirements
------------------
- Python 3.6
- Django 3.1
- Django REST Framework

## Installation
------------------
After you cloned the repository, you want to create a virtual environment, so you have a clean python installation.
You can do this by running the command
```
python -m venv env
```

After this, it is necessary to activate the virtual environment, you can get more information about this [here](https://docs.python.org/3/tutorial/venv.html)

```
source env/bin/activate
```

You can install all the required dependencies by running
```
pip install -r requirements.txt
```

## Structure
-----------------
In a RESTful API, endpoints (URLs) define the structure of the API and how end users access data from our application using the HTTP methods - GET, POST, PUT, DELETE. Endpoints should be logically organized around _collections_ and _elements_, both of which are resources.

In our case, we have one single resource, `movies`, so we will use the following URLS - `/movies/` and `/movies/<id>` for collections and elements, respectively:

Endpoint |HTTP Method | CRUD Method | Result
-- | -- |-- |--
`movies` | GET | READ | Get all movies
`movies/:id` | GET | READ | Get a single movie
`movies`| POST | CREATE | Create a new movie
`movies/:id` | PUT | UPDATE | Update a movie
`movies/:id` | DELETE | DELETE | Delete a movie

## Use
-------------------
We can test the API using [curl](https://curl.haxx.se/) or [httpie](https://github.com/jakubroztocil/httpie#installation), or we can use [Postman](https://www.postman.com/)

Httpie is a user-friendly http client that's written in Python. Let's try and install that.

You can install httpie using pip:
```
pip install httpie
```

First, we have to start up Django's development server.

```
python manage.py makemigrations
python manage.py migrate
```
then, we add movie details (add multiple entries) using following command. This creates a table in **sqlite** database :
```
python manage.py shell
from movies.models import Movie
from django.contrib.auth.models import User

movie1 = Movie(title='Avengers : Endgame', genre='Action', year=2019, creator=User.objects.get(username='PERSON123'))
movie1.save()

```

Finally, we run the server to view the results : 
```
python manage.py runserver
```
Only authenticated users can use the API services, for that reason if we try this:
```
http  http://127.0.0.1:8080/api/v1/movies/
```
we get:
```
{
    "detail": "Authentication credentials were not provided."
}
```

Instead, if we try to access with credentials (see later on how to get access tokens):

```
http http://127.0.0.1:8000/api/v1/movies/1/ "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjk0Njc0MTUzLCJqdGkiOiI5Y2NlMGVjYTljNzA0ODAxYjdlYWY0NGNmMTZiNWMxMCIsInVzZXJfaWQiOjF9.q71LB8606989AYD6ObOtq-jc0jFzur80XsB96fJ2vaY"
```
we get the all the movie as :

![All Movies](https://github.com/kunalkumar168/Creating-RESTFull-API-with-CRUD-application-in-Django-REST-Framework/blob/main/images/AllMovies.png?raw=true)

If we want to see just one movie out of all, we can do :

![One Movies](https://github.com/kunalkumar168/Creating-RESTFull-API-with-CRUD-application-in-Django-REST-Framework/blob/main/images/Movies1.png?raw=true)

## Create users and Tokens
----------------------

First we need to create a user, so we can log in
```
http POST http://127.0.0.1:8000/api/v1/auth/register/ email="example@email.com" username="PERSON123" password="PASS@123" password2="PASS@123" first_name="first" last_name="last"
```

After we create an account we can use those credentials to get a token

To get a token first we need to request
```
http http://127.0.0.1:8000/api/v1/auth/token/ username="PERSON123" password="PASS123WORD"
```
after that, we get the token
```
{
    "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjk0NjcyNjc5LCJqdGkiOiI4MzM0MzQwZWJmNDY0ZjYwOTUzZWVmNzJmN2NkMDQ0MyIsInVzZXJfaWQiOjF9.k6HiHJAIXxjDgG9qRNdmqS7gvZObBVCS0vwB8ejR4Ms",

    "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTY5NDc1ODc3OSwianRpIjoiNjZlMTEwZjZjZTUwNDk2MWJlOWNlYmFjNTlhMDY0MTQiLCJ1c2VyX2lkIjoxfQ.dMkBuxo8WQSe1zfFRHpxc2fWr5Zo2p0AMoW9lZ4Ihqo"
}
```
We got two tokens, the access token will be used to authenticated all the requests we need to make, this access token will expire after some time.
We can use the refresh token to request a need access token.

requesting new access token
```
http http://127.0.0.1:8000/api/v1/auth/token/refresh/ refresh="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTY5NDczODIzNSwianRpIjoiNTlkMTk1NmIyNGI4NGQzNWI0OWJiZTc4MDUzNmRlOTciLCJ1c2VyX2lkIjoxfQ.tMul0rXLxJr5r9sB3fDKJ699kAgW_T6nl0OZoMEgqhc"
```
and we will get a new access token
```
{
    "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjk0NjUyMTM1LCJqdGkiOiIyZjgxNTZkNDc5MzI0OTBhODEwNjM5YmJmYTQ2Nzc0YSIsInVzZXJfaWQiOjF9.8A5UAUtuJ-xtoUjYi18W2R3l-HqSyfBtDmTyP8FoFNY"
}
```


The API has some restrictions:
1.   The movies are always associated with a creator (user who created it).
2.   Only authenticated users may create and see movies.
3.   Only the creator of a movie may update or delete it.
4.   The API doesn't allow unauthenticated requests.

## Commands
----------------------

| Description          | HTTP Command                                                                                           |
|----------------------|--------------------------------------------------------------------------------------------------------|
| Get all movies       | `http http://127.0.0.1:8000/api/v1/movies/ "Authorization: Bearer {YOUR_TOKEN}"`                     |
| Get a single movie   | `http GET http://127.0.0.1:8000/api/v1/movies/{movie_id}/ "Authorization: Bearer {YOUR_TOKEN}"`       |
| Create a new movie   | `http POST http://127.0.0.1:8000/api/v1/movies/ "Authorization: Bearer {YOUR_TOKEN}" title="Ant Man and The Wasp" genre="Action" year=2018` |
| Full update a movie  | `http PUT http://127.0.0.1:8000/api/v1/movies/{movie_id}/ "Authorization: Bearer {YOUR_TOKEN}" title="AntMan and The Wasp" genre="Action" year=2018` |
| Partial update a movie | `http PATCH http://127.0.0.1:8000/api/v1/movies/{movie_id}/ "Authorization: Bearer {YOUR_TOKEN}" title="AntMan and The Wasp"` |
| Delete a movie       | `http DELETE http://127.0.0.1:8000/api/v1/movies/{movie_id}/ "Authorization: Bearer {YOUR_TOKEN}"`     |


## Pagination
----------------------

The API supports pagination, by default responses have a page_size=10 but if you want change that you can pass through params page_size={your_page_size_number}

Pages | HTTP Command
-- | -- 
Page 1 | `http http://127.0.0.1:8000/api/v1/movies/?page=1 "Authorization: Bearer {YOUR_TOKEN}"`
Page 3 | `http http://127.0.0.1:8000/api/v1/movies/?page=3 "Authorization: Bearer {YOUR_TOKEN}"`
Page 3 and Page 15 | `http http://127.0.0.1:8000/api/v1/movies/?page=3&page_size=15 "Authorization: Bearer {YOUR_TOKEN}"`

Sample output :
![Pagination](https://github.com/kunalkumar168/Creating-RESTFull-API-with-CRUD-application-in-Django-REST-Framework/blob/main/images/Pagination.png?raw=true)

## Filters
----------------------

The API supports filtering, you can filter by the attributes of a movie like this

Filter | HTTP Command
-- | --
Movie filter | `http http://127.0.0.1:8000/api/v1/movies/?title="Avatar" "Authorization: Bearer {YOUR_TOKEN}"`
Year filter | `http http://127.0.0.1:8000/api/v1/movies/?year=2016 "Authorization: Bearer {YOUR_TOKEN}"`
Year Range filter | `http http://127.0.0.1:8000/api/v1/movies/?year__gt=2019&year__lt=2022 "Authorization: Bearer {YOUR_TOKEN}"`
Genre filter | `http http://127.0.0.1:8000/api/v1/movies/?genre="Action" "Authorization: Bearer {YOUR_TOKEN}"`
Username filter | `http http://127.0.0.1:8000/api/v1/movies/?creator__username="myUsername" "Authorization: Bearer {YOUR_TOKEN}"`

Sample output :
![Filter](https://github.com/kunalkumar168/Creating-RESTFull-API-with-CRUD-application-in-Django-REST-Framework/blob/main/images/Filter.png?raw=true)


