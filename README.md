## Preface

> The goal of this session is to provide an overview on Building APIs.

## Bootcamp Project

### Intro

> Learning Goals

1. Build a RESTful HTTP API
2. Become familiar with the Python language and Falcon web framework
3. Use a ORM (object relational mapper) to interact with a SQL database
4. Understand the basics of unit testing your code

> Procedural

1. Please join the slack channel #bootcamp_group_project_2021
2. During presentation please use slack for any questions
3. If you need immediate help use the raise hand reaction in Zoom
4. You may use any resource - google, stack overflow, each other, but we reccomend you start with the documentation

> Documentation & Useful Links

- Python - https://docs.python.org/3/
- PeeWee Database ORM - http://docs.peewee-orm.com/en/latest/index.html
- Falcon Web Framework - https://falcon.readthedocs.io/en/stable/
- RESTful HTTP API Design - https://www.restapitutorial.com/index.html
- OpenAPI aka Swagger - https://swagger.io/specification/
    - Swagger Editor - https://editor.swagger.io/

### Run Locally

```
git clone <this repo>
cd 2021Bootcamp-API
make start
```
Let's make sure the API works:
- Navigate to `localhost:8000/heartbeat` in a web browser
- You should see 'Hello World! You did it!'

Now lets try out a few other commands
```
make test
make logs
```
`make test` will show a bunch of tests failing. That's ok, by the time you are done all tests should be passing.
`make logs` will show a constant stream of the API's server logs. Try going to localhost:8000/heartbeat again and refresh the page a few times. Every refresh should show a new entry in the log. Next let's verify that the database is functioning:
- Navigate to `http://localhost:8000/v1/products/1`

You should see `{"id": 1, "name": "Standard SSL", "description": "Your standard SSL certificate", "image_url": null, "price": 14.99, "is_on_sale": false, "sale_price": 8.99}`. The data for this product was read out of a PostgreSQL database running separately from the API.

### OpenAPI Specification AKA Swagger

Let's talk about Swagger now! Swagger is a widely used Framework to design APIs and generating automatic documentation for them. From Swagger.io,
> Swagger allows you to describe the structure of your APIs so that machines can read them. The ability of APIs to describe their own structure is the root of all awesomeness in Swagger. Why is it so great? Well, by reading your API’s structure, we can automatically build beautiful and interactive API documentation.

All we need to do is add specifications about the API in a yaml or json file (based on OpenAPI Specs), and swagger would generate and interactive documentation which you can use to even do automated testing.

#### Add your API specification to the project

You should have created an API specification during your API training and tools session. If you do not have one a trainer can provide one for you.

- Replace the 2021Bootcamp-API/swagger/api.json with your API specification
- Navigate to your API root: `http://localhost:8000/`
- You should see interactive documentation for your UI
- If you have errors, try switching to the [swagger editor](https://editor.swagger.io/) for debugging.
- Try interacting with the API via swagger
    - click on `/products​/{id}` and then `Try it out`
    - Fill in the required product_id with `1` and click `Execute`
    - Below you should see the request made and a response of 200 with some data


### So what is the fuss behind Falcon?

![That's a big question](https://media.giphy.com/media/W5Ub2lhJPWlL4iXnNL/giphy.gif)

According to the official Falcon documentation (https://falconframework.org/),

> When it comes to building HTTP APIs, other frameworks weigh you down with tons of dependencies and unnecessary abstractions. Falcon cuts to the chase with a clean design that embraces HTTP and the REST architectural style.

Falcon is a light-weight bare-metal web API framework aimed at building very fast backends. Its simple and has a
non-opinionated way of doing things which results in a flexible codebase.

It just makes building APIs **_fast, easy and flexible_**.

In contrast a "heavy-weight" or "full" framework might have components for templates, front end webpages, authentication, and more. However we are only building an API so we do not need all of that.

So now that we know what Falcon is, lets start with going over the project setup.

### Project Setup

There are four main components to the project setup:

1. Dockerfile
2. docker-compose.yml
3. requirements.txt
4. Makefile

The necessary packages required to build a Falcon API are:

1. Falcon (for the API)
2. Gunicorn (for the app server)
3. Requests (for HTTP requests)
4. PyTest (for writing unit test cases for the API)
5. Psycopg2 (for adding a local Postgres SQL database for the API)
6. Swagger (for building a swagger ui for the API)
7. Coverage (for checking code coverage of the tests for the API)

Docker does the heavy lifting of installing the required packages for the project and provide you an isolated
environment where you can build and test your changes.

> To check for docker logs, you can always check the list of containers running and then use docker logs command to tail the logs. For example,
> `docker ps` # This would give you a list of the containers
> `docker logs container-name` # This would tail the container logs

You can then use commands in makefile to run your project. The following ones would come in handy.

> To run the api, postgres, and the tests in the background, use the following command
>```python
>make start
>```

> Kill the container using the following make command
> ```python
> make stop
>```

> Check on the container using the following make command
>```python
> make status
> ```

For a complete list of make commands, run
```python
make help
```

Its time to dive into the code!

### Resources

We want to build an API capable of exposing Cart information. Falcon uses the concept of resources borrowed from REST
architectural style.

> In a nutshell, resources are all things on the API which can be accessed via a URL.

So, if we want to use the API to fetch us product information or cart items, or even add, update or delete any product,
we can use the API.

These resource names are defined as python classes and act as controllers and handle the response to a request for that
resource.

The resources are listed in the routes folder in cart_api. Let's go through one of those for products.

```python
class Product:
    def on_get(self, req, resp, product_id):
        product = DatabaseProducts.get(id=product_id)
        resp.media = model_to_dict(product)
        resp.status = falcon.HTTP_200
```

The above code snippet defines a resource `Product` containing functions for the HTTP requests (more on that later).

> The above resource can be used to perform operations related to Product data only.

But wait, where is the data being stored?!

![Where is the data](https://media.giphy.com/media/Y4KWPRcaY1xuZgmwRY/giphy.gif)

### Database

The project uses a local Postgres SQL database to store data tables. Python has a library called `peewee` which is a ORM
for bridging the data stored in relational tables to Python objects. The official documentation can be found here: https://docs.peewee-orm.com/en/latest/peewee/api.html

ORMs make it much easier to interact with a database, no SQL query language needed ! They are tied to concepts of object oriented programming. The Model is a special type of class which defines the schema of a database table and instances of that class are the rows or specific records of data.

PeeWee makes it really simple to define data models and data tables. Let's start dissecting the `database.py` file. There
are three main components to it:

```python
database = os.environ.get("POSTGRES_DB", "bootcamp")
user = os.environ.get("POSTGRES_USER", "bootcamp")
password = os.environ.get("POSTGRES_PASSWORD", "bootcamp")
hostname = os.environ.get("POSTGRES_HOST", "localhost")

from playhouse.postgres_ext import *

ext_db = PostgresqlExtDatabase(database, user=user, password=password, host=hostname)


class BaseModel(Model):
    class Meta:
        database = PostgresqlDatabase(database,
                                      user=user,
                                      password=password,
                                      host=hostname,
                                      autorollback=True)
```

First we establish a method of connecting to the database. In the above code snippet, we start with extracting the database credentials stored as environment variables (They are passed from the docker-compose file). Once we have the values, we create a postgres extension object using the
credentials (they would be used later for binding). We also define our base data model containing our database object in
the Meta class. These base models are then extended into table models.

```python
class DatabaseProducts(BaseModel):
    id = AutoField(primary_key=True)
    name = CharField()
    description = CharField(null=True)
    image_url = CharField(null=True)
    price = DoubleField()
    is_on_sale = BooleanField()
    sale_price = DoubleField(null=True)

    @classmethod
    def prepopulate(cls):
        products = [DatabaseProducts(id=1, name="Standard SSL", description="Your standard SSL certificate", price=14.99,
                             is_on_sale=False, sale_price=8.99),
                    DatabaseProducts(id=2, name="Wildcard SSL", description="Encrypt any subdomains may exist on the site",
                             price=29.99, is_on_sale=True, sale_price=19.99),
                    DatabaseProducts(id=3, name="Domain - .com", description="Purchase a .com domain", price=9.99,
                             is_on_sale=False),
                    DatabaseProducts(id=4, name="Domain - .org", description="Purchase a .org domain", price=8.99,
                             is_on_sale=False),
                    DatabaseProducts(id=5, name="Domain - .co", description="Purchase a .co domain", price=8.99,
                             is_on_sale=True, sale_price=4.99)]
        DatabaseProducts.bulk_create(products)
```

Second, we create our first model (an extention of the base model) which corresponds to the database Product
table. Each column for the table has a corresponding SQL storage class (such as varchar, int, etc.)
We also utilise the `prepopulate` function in peewee to add rows to our database table.

> There is a special field here for representing the primary key. See http://docs.peewee-orm.com/en/latest/peewee/models.html#primary-keys-composite-keys-and-other-tricks for more info.

Finally, the remaining code in database.py is making sure that the tables are binded to the DB, created and populated.

If you compare your API spec you may notice while we have a Product model, there is not yet a model for CartItems

> **Exercise 1**: Create a new model for the resource `CartItem`. Optionally prepopulate with any number of rows. _Depending how you design your CartItem model you may need to update the EXAMPLE_CART_ITEM in cart_api_tests/test_exercises.py_

The model should match your API specification. Make sure the fields and their data types are consistent with your swagger. If you are unsure what fields to use or how to proceed try checking the PeeWee documentation on Models and Fields: http://docs.peewee-orm.com/en/latest/peewee/models.html

If successful you should be able to run `make test` and see that `Exercise1::test_import_model` is now passing

### HTTP Methods

Okay, now that we have a functioning database, lets talk about those functions in the Resource class.

The functions listed in the resource class are HTTP methods often referred to as "responders". Each API request is
mapped to these methods following the `on_*()` convention, where * is any one of the standard HTTP methods lowercased.

Each responder takes (at least) two params, one representing the HTTP request, and one representing the HTTP response to
that request. By convention, these are called req and resp, respectively. The responder may also accept a parameter for any variables which are present in the route.

```python
class Product:
    def on_get(self, req, resp, product_id):
        product = DatabaseProducts.get(id=product_id)
        resp.media = model_to_dict(product)
        resp.status = falcon.HTTP_200

    def on_delete(self, req, resp, product_id):
        DatabaseProducts.delete_by_id(product_id)
        resp.status = falcon.HTTP_204
```

In the above snippet, we have defined two HTTP operations. The first one is a controller for a `GET` request which would
try to fetch the Product information based on the `product_id` (passed in from the request URL) and then send it off as part of the response.

The second one is a `DELETE` request which would delete the item.

The query operations such as `.get` or `.delete_by_id` are directly done on the tables created in the database.py file
and importing those tables classes in this file. For example,

```python
from cart_api.database import Products as DatabaseProducts
```

the above snippet imports the Database class `Products` and does operations on the related resource.

Also, Notice the HTTP response codes set in each of these controllers. These help to identify whether a API request was
successful or not.

> For a complete list of falcon response codes, visit https://falcon.readthedocs.io/en/stable/api/status.html
>
> For a complete list of querying functions in peewee, visit here: https://docs.peewee-orm.com/en/latest/peewee/querying.html

### Routing - Let's tie it altogether

Once we have our resources and database ready, all's left to do is instantiate our API and define it's routes.

We start with creating an instance of the Falcon API and the resource class in `api.py`

```python
api = falcon.API()
product = Product()
```

And make product callable by adding route to the Facon API

```python
api.add_route('/v1/products/{product_id:int}', product)
```

> Notice the “/” preceding the “products” . Do not forget this, all URLs in falcon should have a root “/”. Also, `product_id:int` indicates that an id of type int needs to be passed into the responder for making the operation on the resource.

And BOOM! We have covered everything it takes to create an HTTP API that serves data from a SQL database.. Now it's time to check it out. You can use make file or the docker compose to run the API.

![Boom](https://media.giphy.com/media/mks5DcSGjhQ1a/giphy.gif)

(The API uses gunicorn, a server in the background for running).

Once running, go to `localhost:8000/v1/products/{id}` in your browser and substitute `id` with an actual id
from the database.
(_Hint: Remember in `database.py` we had pre-populated that cart item table. Use an id from those values._)

After hitting enter, you should see a JSON response with the entire row.

Or even better, use SWAGGER for the API. Go to `localhost:8000/` and navigate to `products` section to make the request.

But wait, how do I create (POST) a new product and what about this `GET /products` that doesn't seem to work?

> **Exercise 2**: Lets create another resource class to do operations on Product table when you do not want to specify an id.
> You can call the resource `Products` (Notice the `s`). The resource should support the following two operations:
> 1. Add a product in a single request using the `on_post` method. The responses should contain the appropriate status and that data it just created.
> 2. Display all the products using `on_get` method
>
_(Hint: The falcon request object should contain media data with all the required columns. Use the Product Model to insert the data into the database)_

- https://falcon.readthedocs.io/en/stable/api/media.html
- https://falcon.readthedocs.io/en/stable/api/request_and_response_wsgi.html
- https://docs.peewee-orm.com/en/latest/peewee/querying.html

Once complete, run `make test` and you should see the Exercise2 tests have now PASSED

### Break Time
![Freedom](https://media.giphy.com/media/QsyVRYGgsO7yc2HU5O/giphy.gif)

> **Exercise 3**: Build the Cart Item resources similar to Product. Create two new resources called `CartItem` and `CartItems` using the `DatabaseCartItem` database Model. The resources should support the following operations. (To be able to GET a CartItem you will need to first create one by implementing POST or using prepopulate in database.py)
>
> CartItem:
> 1. Fetch a Cart Item row based on the given item_id
> 2. Delete a Cart Item row based on the given item_id
> 3. Update a Cart Item row based on the given item_id
>
> CartItems:
> 1. Add a new Cart Item row
> 2. List out all the Cart Item rows
>
> Hints: Do not forget to add routes for the new resources to the Falcon API class. Several tests require POST to be correct before they will pass

Useful links:
- https://www.restapitutorial.com/lessons/httpmethods.html
- https://docs.peewee-orm.com/en/latest/peewee/querying.html
- https://falcon.readthedocs.io/en/stable/api/request_and_response_wsgi.html
- https://falcon.readthedocs.io/en/stable/api/status.html

Now that all tests are passing you should see a code coverage report for the unit tests

**Congratulations!!!**

![You did it](https://media.giphy.com/media/3otPoS81loriI9sO8o/giphy.gif)

### Bonus: Adding Tests for the API

Adding test cases is paramount to any software application. It validates the Software application and ensures that it is
functional and reliable.

![Testing is important](https://media.giphy.com/media/l0MYAY18Pxyxwu2xa/giphy.gif)

Testing our API will ensure that it is doing what it is supposed to do and help us catch any bugs early on during the
development process. We can use the API response object and status codes to create test cases for the resources.

For the purpose of this project, we would be using Python's unittest library to write test cases and generate a coverage
report!

Take a look at the test_cartitems.py file in the cart_api_tests folder. It lays down some tests for the `CartItem`
and the `CartItems` resources. Let's understand what is happening there.

```python
CARTITEMS_PATH = "/v1/cartitems"
CARTITEM_PATH = "/v1/cartitems/{item_id}"


class Exercise3(TestClient):

    def test_get_items(self):
        response = self.simulate_get(CARTITEMS_PATH)
        self.assertEqual(response.status_code, 200)
        body = response.json
        self.assertIsInstance(body, list)

    def test_get_item(self):
        body = dict(name="Standard SSL", product_id=1, price=4.99, quantity=1)
        response = self.simulate_post(CARTITEMS_PATH, json=body)
        self.assertEqual(response.status_code, 201)
        self.aitem = response.json

        response = self.simulate_get(
            CARTITEM_PATH.format(item_id=self.aitem["id"])
        )
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json["name"], self.aitem["name"])
```

In the Exercise3 class, each function represents contains test cases to test a singular HTTP method for the resources. Ideally, you would want to have at least one test function per HTTP method per resource.
Since the resources `CartItem` and `CartItems` have the same underlying table, we have combined them into one test class.

Each of the function is doing the API call with the right combination of your API URI (i.e. localhost:8000) and the route for that resource (e.g. /v1/cartitems).
The API response and the response code is then used against `assert` statements to check if the value is valid or expected. The full list of assert statements can be found here: https://docs.python.org/3/library/unittest.html

Let's run the above test cases using the following commands and see what happens.

`make test`

From the output, it is evident that the test cases are being executed and we are seeing incomplete coverage. This is due to the fact that we don't have test cases to cover each method for each resource.

### Bonus Exercises
The following exercises will not be required for the next phases of the project, but if you want to challenge yourself and improve your python, testing, and API design skills then see how far you can get. Reach out on slack if you need extra help or guidance.

> **Bonus Exercise 1** : 100% code coverage - Add some test(s) of your own to test_bonus_exercises.py. Identify what needs to be tested by looking at the missing lines in the coverage report and write some test(s) that will cover them.

Code coverage, while not perfect is the foundational metric for code quality. Not only is 100% coverage a badge of honor, but it will give you the confidence to add features or refactor without fear of breaking things.

> **Bonus Exercise 2** : Don't allow duplicate products - The product manager for WeResellAllTheThings.com called with a complaint: There are a bunch of products in the database with the same name and he would like the API to prevent that from happening.

- Make a code change that would prevent any product from being created if there is an existing product with same name
- Think about how to communicate to the caller that they have made a bad request
- This is likely to affect your code coverage, make sure you add any new tests you need to get 100% again
- The API spec for POST products shows the only possible response is a 201: successful create, is this still accurate?

> **Bonus Exercise 3** : Requesting a product/item that does not exist should return a status 404 - Using swagger or the browser try to view a product number which you know does not exist, you will get an unhandled Internal Server Error, status 500. Use `make logs` and do it again and you will even see an exception stack trace for your error.
- One way to show off your bug-fixing skills is to use [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development). In short try to write a test first that shows there is a problem before you start writing code to fix it.
- One way to handle this is with python exception handling: https://docs.python.org/3/tutorial/errors.html#handling-exceptions
- Another way is to use Falcon's error handling, there is actually an example of it in api.py. Documentation: https://falcon.readthedocs.io/en/stable/api/app.html#falcon.App.add_error_handler
- Regardless of the method it is ideal to capture the specific error that is being raised, don't be afraid to google for help on this one.
- Make sure your code coverage is still 100% after making this change.
## Outro

As evident from the above exercise, building an API in Falcon is super simple, flexible and fast!

### References
- https://falcon.readthedocs.io/en/stable/#
- https://docs.peewee-orm.com/en/latest/peewee/api.html
- https://medium.com/@gaurav_52429/developing-rest-apis-in-under-50-lines-of-code-using-falcon-in-python-25d3b47a493d
- https://simpleprogrammer.com/api-testing/
- https://swagger.io/
- https://docs.docker.com/
