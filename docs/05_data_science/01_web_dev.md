# 1. Web Development



## Table of Contents

1. Overview
2. Tools
3. Architecture
4. Back-End
5. Front-End Templates
6. Models
7. Testing
9. Exercise

## 1. Overview

In this part of the workshop, we'll learn about how to get started creating web applications that we'll later fill in with 
generated data to create a proof of concept, or maybe even the real deal. Before we begin though, let's talk about what 
microservices are and how to use them.

**What are microservices?**
Microservices are an architectural approach to building applications as a collection of small and modular, 
independently deployable services.

**What are machine learning microservices?**
Machine learning microservices apply this approach to ML systems by decomposing them into smaller 
services that each focus on a specific capability or model. Some examples of machine learning microservices:
- Model Training Service - Handles training ML models on new data.
- Model Serving Service - Deploys trained models and provides predictions/inferences. 
- Data Processing Pipeline - Microservices for data ingestion, cleaning, preprocessing.
- Model Monitoring Service - Tracks model performance and drift.
- Experiment Tracking Service - Logs model experiments and results.

The benefits of using microservices for machine learning include:

- Independent scaling - Can allocate more resources to demanding services.
- Fault isolation - If one service fails, others are not affected.
- Flexible deployment - Can rapidly deploy updates to individual services.
- Polyglot support - Mix languages/frameworks within services.
- Organizational alignment - Teams can own discrete services.

The main challenges are the added complexity of distributed systems and the need for coordination 
between services. Clear communication protocols and well-defined APIs are essential.

Overall, microservices enable faster iteration and more robust and resilient ML systems, but require 
more up-front design and infrastructure coordination.

Let's go over the tools we'll be using in this section.

## 2. Tools

For this section, we will be using the following tools.

- FastAPI: FastAPI is like having a team of skilled architects and builders for constructing a 
house quickly and efficiently. It provides clear blueprints (API endpoints) and customization 
options, ensuring rapid development of robust and personalized APIs.
- HTML: HTML is a markup language for building websites.
- Tailwind CSS: a powerful CSS framework for building responsive websites.
- Jinja2: a templating tool that allows you to write HTML templates.

## 3. Architecture

![archi1](../images/architecture_1.png)

Every application or system needs an architecture, and even thought these don't need to be built into a diagram, it is 
good practice to visualize how we want the system to look like and function before we get to coding. Let's start there.

We will need:
- a back-end
- a front-end
- 2 machine learning applications
- tests
- A DataBase (optional for this tutorial)

## 4. The Back-End

Here's an attempt to describe the front-end and back-end of a microservices application using analogies:

The front-end is like a traveler exploring a foreign city, navigating between sites and activities. It 
acts as the user interface, calling different services to assemble experiences. The React/Vue UI is the 
traveler's map, guiding them between locations. Redux/Flux stores are travel journals, recording visits 
to services. APIs are transit systems, with protocols like GraphQL as subway maps. User auth is visa 
security, granting access privileges.

The back-end is like a bustling marketplace, full of vendors running independent shops. Services are 
merchant stalls, focused on specific capabilities. Data pipelines act as supply chains, moving inventory 
between stalls. Monitoring services are the market inspector, checking goods and stall conditions. APIs are 
the signboards and directions that connect the marketplace. Scaling changes the number of vendor stalls. New 
capabilities are added by launching new pop-up shops.

To travel the market (use the app), the front-end explorer (UI) relies on the directions (APIs) to visit 
the right merchants (services). Back-end organization and protocols enable smooth exploration. Microservices 
create a thriving software bazaar!

Let's begin with an example server that has one kind of API. Every time we call our API we'll get a joke back.


```python
%%writefile ../src/examples/jokes.py

from fastapi import FastAPI
import pyjokes

app = FastAPI()

@app.get("/joke")
def get_joke():
    joke = pyjokes.get_joke()
    return {"joke": joke}
```

To run a server with FastAPI we need to use uvicorn or gunicorn. The reason for this is that FastAPI 
applications need a specialized ASGI (Asynchronous Server Gateway Interface) server to run and to handle 
concurrent connections efficiently. ASGI servers can handle asynchronous request processing, allowing 
multiple requests to be processed simultaneously without blocking the execution flow, which is of high 
importance for high-performance web applications, APIs, and services.

Uvicorn and Gunicorn are popular ASGI servers used to run FastAPI applications:

To run a FastAPI application using Uvicorn, you can use the following command:
   
```
uvicorn main:app --reload
```

Here, `main` refers to the Python file (`main.py` in this case), and `app` is the instance of our FastAPI application.

Gunicorn is a production-ready ASGI server that can handle high loads and is suitable for deploying 
demanding applications. Gunicorn provides more configuration options and allows you to scale your 
application across multiple worker processes or even multiple server instances behind a load balancer

To run a FastAPI application using Gunicorn, you can use the following command:
   
```
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

In this command, `-w 4` specifies the number of worker processes (you can adjust this based on your 
server's resources), and `main:app` refers to the module and FastAPI instance.

With that bit of intro out of the way, let's run our app in the terminal.

```sh
uvicorn main:app --reload
```

Once our server is up and running, we can send a GET requests to it using the requests library as below.


```python
import requests

response = requests.get("http://localhost:8000/joke")
print(response.status_code, "\n", response.json())
```

Please keep your expectations low and your dad jokes tolerance high with these examples. The 
`pyjokes` library can be a hit and miss with the quality of the jokes.🫣

One of the cool features of FastAPI is that comes with support for [swagger documentation](https://swagger.io/docs/). 
This means that the kinds of requests your users can make to your microservices built with FastAPI 
will be readily available at the `http:localhost:8000/docs` endpoint. For example,

![swag_1](../images/swagger1.png)

and you also get visibility on each method.

![swag_2](../images/swagger2.png)

### Exercise

Download a package a create a new API that, when called, returns something back. You can get as 
creative as you'd like to with the result. 😎

Here are some examples:
- `PyDictionary`
- `pyqrcode`
- `pyshorteners`
- `auto_corrector`


```python

```


```python

```

Now that we know a bit about how we can create APIs, let build the back-end of our first application.


```python
%%writefile ../main.py

from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/")
async def read_root(request: Request):
    pass

@app.get("/service1")
async def read_page1(request: Request):
    pass

@app.get("/service2")
async def read_page2(request: Request):
    pass
```

Although the file above needs a bit of modification, it already contains a good blueprint for our backend. Let's 
unpack the file above.

In the snippet above, we created an application with 3 components:
1. A home route
2. A page with some service
3. Another page with another service

These 3 pieces will come up as `your_kul_website.com` (the home page), `your_kul_website.com/service1`, 
and `your_kul_website.com/service2`.

Each of these services will have a front-end template, and the functionality within each 
could be composed of multiple services as well (as we will see shortly).

## 5. Front-End Templates

As data professionals or machine learning engineers, chances are that we might not have much 
experience with front-end development, but that can't and shouldn't stop us from being able to put 
some makeup on our apps. To help us with this, we have Jinja2, a Python library that helps us 
build templates that add structure and format to the content of our applications.

Let's build a template for the home page of our web app. This page will contain 
- a title
- a description
- a menu
- and blocks to hold the content we'll create with our generative AI models

plus a nice look and feel. We will add it to a templates directory step-by-step.

### 5.1 The Home Page


```python
%%writefile ../templates/home.html

<!DOCTYPE html>
<html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Creativity as a Service with Machine Learning</title>
        <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    </head>

    <body class="bg-gray-100 p-8">
        <div class="text-4xl font-bold mb-8 text-center">Creativity as a Service with Machine Learning</div>
    </body>

</html>
```

What we have above is a standard HTML file with
- a type --> `DOCTYPE` which tells a web browser that the type of that document is HTML
- Container tags
  - `<html>`
    - a `<head>` --> you can think of this as the settings of the document
    - a `<body>` --> this has all of the content of the document
- and some style pulled from tailwind.css

Now that we have a template, we can update our API one step at a time and initialize our 
service. If the previous server we instantiated (`jokes.py`) is still running, make sure to 
stop it first with Ctrl + C in your terminal.


```python
%%writefile ../main.py

from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates

app = FastAPI()

templates = Jinja2Templates(directory="./templates")

@app.get("/")
async def read_root(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})
```

Notice that FastAPI comes with a handy class to tell it where our templates live. Once we point 
it to the right directory, we can write the name of the template we want that route to use and 
send it to users when they request it.

In your terminal, run the following command:

```bash
uvicorn main:app --reload
```
and then open the browser at `http://localhost:8000/`. You should be able to see the following home page.

![hp](../images/home_screen.png)

Notice that there is some separation of concerns happening here. If could, realistically, have a team 
members asynchronously focusing on model development, others would be working on these templates, and 
others on the back-end of our desktop app, website, mobile app, game, edge device, etc.

### 5.2 Adding a Menu

Now that our website has a title, let's create the menu and a description for it.


```python
%%writefile ../templates/home.html

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Creativity as a Service with Machine Learning</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>

<body class="bg-gray-100 p-8">
    <div class="text-4xl font-bold mb-8 text-center">Creativity as a Service with Machine Learning</div>

    <!-- Navigation Menu -->
    <div class="flex justify-center mb-8">
        <a href="#" class="mx-4 px-6 py-2 bg-blue-500 text-white rounded-full hover:bg-blue-700 transition duration-300">Home</a>
        <a href="#service1" class="mx-4 px-6 py-2 bg-green-500 text-white rounded-full hover:bg-green-700 transition duration-300">Service 1</a>
        <a href="#service2" class="mx-4 px-6 py-2 bg-indigo-500 text-white rounded-full hover:bg-indigo-700 transition duration-300">Service 2</a>
    </div>

    <div class="text-lg mb-8 text-center">
        Unlock your creativity with our innovative machine learning services. Explore a world of possibilities where technology meets artistry.
    </div>


</body>

</html>
```

If you refresh the page, you will now see the following output:

![svc2](../images/home_screen2.png)

Let's create now a first service and call it `page1.html`.


```python
%%writefile ../templates/page1.html

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Machine Learning Microservice</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.16/dist/tailwind.min.css" rel="stylesheet">
    <style>
        body {height: 100vh; display: flex; flex-direction: column; justify-content: center; align-items: center;}
        .container {text-align: center;}

        #launchButton {display: none;}
        #launchButtonLabel {
            cursor: pointer; background-color: #4CAF50; color: white; padding: 14px 32px; text-align: center; text-decoration: none;
            display: inline-block; font-size: 16px; margin-top: 20px; border-radius: 8px; transition: background-color 0.3s ease;
        }
        #launchButton:checked+#launchButtonLabel {background-color: #45a049;}
        #gradioIframe {width: 80vw; height: 80vh; border: 1px solid #ccc; border-radius: 10px; display: none; margin-top: 20px;}
        #launchButton:checked~#gradioIframe {display: block;}
    </style>
</head>
<body>
    <div class="container bg-white p-8 rounded shadow-md w-full text-center">
        <h1 class="text-3xl font-bold mb-6">Machine Learning Microservice 1</h1>
        <p class="text-gray-600 mb-8">This microservice provides access to a powerful machine learning model.</p>
        <input type="checkbox" id="launchButton" class="hidden">
        <label for="launchButton" id="launchButtonLabel">Launch ML App</label>
        <div id="gradioIframe" class="mt-6 hidden">
            <iframe src="https://prodia-fast-stable-diffusion.hf.space" frameborder="0" class="w-full h-full"></iframe>
        </div>
    </div>
</body>
</html>
```

Next, we'll need to tell FastAPI how to get ahold of the template we just created. We'll do so by adding a new get method.


```python
%%writefile -a ../main.py

@app.get("/service1")
async def read_page2(request: Request):
    return templates.TemplateResponse("page1.html", {"request": request})
```

We can examine the new template at `http://localhost:8000/service1` without having to restart our service.

While there is nothing to see yet when we click the button to show a machine learning app, we will have a app 
there very soon. 😎 

Finally, we can add the containers to our home template and finish it, for now. 😌


```python
%%writefile ../templates/home.html

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Creativity as a Service with Machine Learning</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>

<body class="bg-gray-100 p-8">
    <div class="text-4xl font-bold mb-8 text-center">Creativity as a Service with Machine Learning</div>

    <!-- Navigation Menu -->
    <div class="flex justify-center mb-8">
        <a href="#" class="mx-4 px-6 py-2 bg-blue-500 text-white rounded-full hover:bg-blue-700 transition duration-300">Home</a>
        <a href="#service1" class="mx-4 px-6 py-2 bg-green-500 text-white rounded-full hover:bg-green-700 transition duration-300">Service 1</a>
        <a href="#service2" class="mx-4 px-6 py-2 bg-indigo-500 text-white rounded-full hover:bg-indigo-700 transition duration-300">Service 2</a>
    </div>

    <div class="text-lg mb-8 text-center">
        Unlock your creativity with our innovative machine learning services. Explore a world of possibilities where technology meets artistry.
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <!-- Container 1 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 1</div>
            <img src="path/to/asset1.jpg" alt="Asset 1" class="mb-2 w-full h-auto">
            <div>Text 1</div>
        </div>
        
        <!-- Container 2 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 2</div>
            <video src="path/to/asset2.mp4" controls class="mb-2 w-full"></video>
            <div>Text 2</div>
        </div>
        
        <!-- Container 3 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 3</div>
            <img src="path/to/asset3.jpg" alt="Asset 3" class="mb-2 w-full h-auto">
            <div>Text 3</div>
        </div>
        
        <!-- Container 4 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 4</div>
            <audio src="path/to/asset4.mp3" controls class="mb-2 w-full"></audio>
            <div>Text 4</div>
        </div>
        
        <!-- Container 5 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 5</div>
            <img src="path/to/asset5.jpg" alt="Asset 5" class="mb-2 w-full h-auto">
            <div>Text 5</div>
        </div>
        
        <!-- Container 6 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 6</div>
            <video src="path/to/asset6.mp4" controls class="mb-2 w-full"></video>
            <div>Text 6</div>
        </div>
        
        <!-- Container 7 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 7</div>
            <img src="path/to/asset7.jpg" alt="Asset 7" class="mb-2 w-full h-auto">
            <div>Text 7</div>
        </div>
        
        <!-- Container 8 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 8</div>
            <audio src="path/to/asset8.mp3" controls class="mb-2 w-full"></audio>
            <div>Text 8</div>
        </div>
        
        <!-- Container 9 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 9</div>
            <img src="path/to/asset9.jpg" alt="Asset 9" class="mb-2 w-full h-auto">
            <div>Text 9</div>
        </div>
        
        <!-- Container 10 -->
        <div class="border p-4 rounded-lg shadow-md">
            <div class="mb-2">Asset 10</div>
            <video src="path/to/asset10.mp4" controls class="mb-2 w-full"></video>
            <div>Text 10</div>
        </div>
    </div>
</body>

</html>
```

If you navigate to our app again, you'll notice that we have empty containers inside the application waiting to be 
filled up. In the next section, we'll start creating content for it and adding it to our app.

## 7. Testing

When testing your FastAPI app with pytest, you can write various types of tests to ensure that 
different aspects of your application are working correctly. Here are some types of tests you can consider:

1. **Unit Tests:**
   - Test individual functions or methods in isolation to ensure they work as expected.
   - For example, you can test the function that fetches jokes from the `pyjokes` library to ensure it returns valid jokes.

2. **Integration Tests:**
   - Test the interactions between different components of your app.
   - For FastAPI apps, this can involve testing how different endpoints interact and whether the data flow between them is correct.

3. **Endpoint Tests:**
   - Test each endpoint of your API to ensure they handle various input scenarios correctly and return the expected responses.
   - Use pytest fixtures to mock HTTP requests and test different HTTP methods (GET, POST, etc.) and request payloads.

4. **Error Handling Tests:**
   - Test how your app handles different types of errors, such as invalid requests or server errors.
   - Ensure that appropriate error responses (with correct status codes and error messages) are returned.

5. **Security Tests:**
   - Test security features, such as authentication and authorization mechanisms.
   - Ensure that unauthenticated users cannot access protected endpoints and that authorized users can access them appropriately.

6. **Performance Tests:**
   - Test the performance of your app by simulating a large number of requests and measuring response times.
   - Identify potential bottlenecks and optimize your code or infrastructure as needed.

7. **Edge Case Tests:**
   - Test your app with edge cases, such as empty inputs, boundary values, or unexpected data formats.
   - Ensure your app behaves correctly and gracefully in these scenarios.

8. **Data Persistence Tests (if applicable):**
   - If your app interacts with a database, test database operations (e.g., CRUD operations) to ensure data integrity.
   - Use fixtures to set up and tear down test data for database-related tests.

When writing tests, consider using pytest fixtures to create reusable setup code for your tests. Also, 
uses the `requests` library in combination with pytest to send HTTP requests to your app's endpoints and validate the responses.

By covering these different aspects of your FastAPI app with tests, you can increase your confidence in its correctness, reliability, and security.

Here are some examples of how you can write tests for your FastAPI app using pytest, covering different testing approaches:

1. **Unit Tests:**

Let's say you have a utility function in a module called `utils.py` that fetches jokes:

```python
# utils.py
import pyjokes

def get_random_joke():
    return pyjokes.get_joke()
```

You can write a unit test for this function:

```python
# test_utils.py
from utils import get_random_joke

def test_get_random_joke():
    joke = get_random_joke()
    assert isinstance(joke, str)
    assert len(joke) > 0
```

2. **Integration Tests:**

For integration tests, you can test the interactions between different components of your app. Here's an example using FastAPI's `TestClient`:

```python
# test_integration.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_get_joke_endpoint():
    response = client.get("/joke")
    assert response.status_code == 200
    data = response.json()
    assert "joke" in data
    assert isinstance(data["joke"], str)
    assert len(data["joke"]) > 0
```

3. **Endpoint Tests:**

You can write tests for specific endpoints, verifying their behavior for different scenarios:

```python
# test_endpoints.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_get_joke_endpoint():
    response = client.get("/joke")
    assert response.status_code == 200
    data = response.json()
    assert "joke" in data
    assert isinstance(data["joke"], str)
    assert len(data["joke"]) > 0

# Add more endpoint tests as needed
```

4. **Error Handling Tests:**

Test how your app handles errors:

```python
# test_error_handling.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_invalid_endpoint():
    response = client.get("/invalid_endpoint")
    assert response.status_code == 404
    assert response.json() == {"detail": "Not Found"}

# Add more error handling tests as needed
```

5. **Security Tests:**

Test authentication and authorization mechanisms (assuming your app has authentication logic):

```python
# test_security.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_authenticated_endpoint():
    # Assuming you have authentication logic and obtain a token
    headers = {"Authorization": "Bearer <your_token>"}
    response = client.get("/authenticated_endpoint", headers=headers)
    assert response.status_code == 200

def test_unauthenticated_endpoint():
    response = client.get("/authenticated_endpoint")
    assert response.status_code == 401
```

These are some initial examples to get you started. Depending on your app's complexity, you might 
need more elaborate tests and additional libraries (such as `pytest-mock` for mocking) to handle 
specific scenarios. Make sure to structure your tests based on your application's architecture and requirements.

## 9. Exercise

1. Find an HTML template online and add it as a new file to the `templates` directory.
2. Find an app that seems interesting to you in the hugging face hub and then copy it to a file 
in the servers directory and tweak it to fit your needs. (e.g. change the name of your app, add 
a different model, or change the layout of the app.)
3. Create a new method for our `main.app` server and add your app to it.
