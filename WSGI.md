# WSGI

##### Chapter 1

Before we dive into the details of WSGI, let's look at what happens when a user uses a web application from a bird-eye's view.

## Web Server

Let's look at the world through a web server's eyes...

Imagine for a moment that you are a web server, like [Gunicorn](https://gunicorn.org/). Your job consists of the following parts:

- You sit around and wait patiently for a request from some kind of a client.
- When a client comes to you with a request, you receive it.
- Then, you take this request to someone called PythonApp and say to him, "Hey dude, wake up! Here's a request from a very important client. Please, do something about it."
- You get a response from this PythonApp.
- You then deliver this response back to your client.

This is the only thing you do. You just serve your clients. You know nothing about the content or anything else. That's why you are so good at it. You can even scale up and down processing depending on the demand from the clients. You are focused on this single task.

## Web App

PythonApp is your software. While a web server should exist and wait for an incoming request all the time, your software exists only at the execution time:

- The web server wakes it up and gives it the request.
- It takes the request and executes some commands on it.
- It returns a response to the web server.
- It goes back to sleep.
- The web server delivers this response back to his client.

The only thing it does is execute when asked to do so.

## The Problem

The scenario above is all good. However, a web server's conversation with PythonApp could have gone a little differently.

Instead of:

"Hey dude, wake up! Here's a request from a very important client. Please, do something about it."

It could have been this:

"Эй, чувак, проснись! Вот запрос от очень важного клиента. Пожалуйста, сделай что нибудь."

Or this:

"Ehi amico, svegliati! Ecco una richiesta da un cliente molto importante. Si prega, fare qualcosa al riguardo"

Or even this:

"嘿，伙计，醒醒吧！这里是一个非常重要的客户端的请求。请做点什么"

Do you get it? The web server could have behaved in a number of different ways and PythonApp needs to learn all these languages to understand what it's saying and behave accordingly.

What this means is that in the past (before WSGI) you had to adapt your software to fit the requirements of a web server. Moreover, you had to write different kinds of wrappers in order to make it suitable across different web servers. Developers generally don't want to deal with such things. They just want to write code.

## WSGI to the Rescue

Here is where WSGI comes in to play. You can think of it as a SET OF RULES for a web server and a web application.

The rules for a web server look like this:

Okay. If you want to talk to PythonApp, speak *these* words and sentences. Also, learn *these* words as well which it will speak back to you. Furthermore, if something goes wrong, *here* are the curse words that PythonApp will say and *here* is how you should react to them.

And the rules for a web application look like this:

Okay. If you want to talk to a web server, learn *these* words because a web server will be using them when addressing you. Also, you use *these* words and be sure that a web server understands them. Furthermore, if something goes wrong, use *these* curse words and behave in *this* way.

## Example

Enough talk, let's fight!

Let's take a look at the WSGI application interface to see how it should behave. According to [PEP 333](https://www.python.org/dev/peps/pep-0333/#the-application-framework-side), the document which specifies the details of WSGI, the application interface is implemented as a callable object such as a function, a method, a class, or an instance with a `__call__` method. This object should accept two positional arguments and return the response body as strings in an iterable.

The two arguments are:

- a dictionary with environment variables
- a callback function that will be used to send HTTP statuses and HTTP headers to the server

Now that we know the basics, let's create a web framework which will take away some market share from [Django](https://www.djangoproject.com/) itself! Our web framework will do something that no one is doing right now: IT WILL PRINT ALL ENVIRONMENT VARIABLES IT RECEIVES. Genius!

Okay, open your text editor of choice, and create that callable object which receives two arguments:

```
def application(environ, start_response):
    pass
```

Easy enough. Now, let's prepare the response body that we want to return back to the server:

```
def application(environ, start_response):
    response_body = [
        f'{key}: {value}' for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)
```

Here, we are iterating over `environ.items()` and making a list of key/value pairs like so:

```
[
    "ENV_VAR_1_NAME": "ENV_VAR_1_VALUE",
    "ENV_VAR_2_NAME": "ENV_VAR_2_VALUE",
    ...
]
```

For example:

```
[
    "USER: Jahongir",
    "LANG: en_US.UTF-8"
]
```

After that, we are joining all the elements of the list `response_body` using `\n` as a line break.

Now, let's prepare the status and headers, and then call that callback function:

```
def application(environ, start_response):
    response_body = [
        f'{key}: {value}' for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)

    status = '200 OK'

    response_headers = [
        ('Content-type', 'text/plain'),
    ]

    start_response(status, response_headers)
```

And finally, let's return the response body in an iterable:

```
def application(environ, start_response):
    response_body = [
        f'{key}: {value}' for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)

    status = '200 OK'

    response_headers = [
        ('Content-type', 'text/plain'),
    ]

    start_response(status, response_headers)

    return [response_body.encode('utf-8')]
```

That's it. Our genius web framework is ready. Of course, we need a web server to serve our application and here we will be using Python's bundled WSGI server.

> Want to learn more about the WSGI server interface? Take a look at it [here](https://www.python.org/dev/peps/pep-0333/#the-server-gateway-side).

Now, let's serve our application:

```
from wsgiref.simple_server import make_server


def application(environ, start_response):
    response_body = [
        '{key}: {value}'.format(key=key, value=value) for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)

    status = '200 OK'

    response_headers = [
        ('Content-type', 'text/plain'),
    ]

    start_response(status, response_headers)

    return [response_body.encode('utf-8')]


server = make_server('localhost', 8000, app=application)
server.serve_forever()
```

Save this file as *wsgi_demo.py* and run it via *python wsgi_demo.py*. Then, go to [localhost:8000](http://localhost:8000/) and you will see all the environment variables listed:

![Headers](https://s3.amazonaws.com/rahmonov.me/post-images/what-is-wsgi-anyway/wsgi-demo.png)

YES! This framework will be very popular!

Now that we know about the WSGI application interface, let's talk about something that we deliberately skipped earlier: Middleware.

## Middleware

Middleware changes things a bit. With middleware, the above scenario will look like this:

- The web server gets a request.
- Now, instead of talking directly to PythonApp, it will send it through a postman (e.g., middleware).
- The postman delivers the request to PythonApp.
- After PythonApp does his job, it gives the response to the postman.
- The postman then delivers the response to the web server.

The only thing to note is that while the postman is delivering the request/response, it may tweak it a little bit.

Let's see it in action. We'll now write a middleware that reverses the response from our application:

```
class Reverseware:
    def __init__(self, app):
        self.wrapped_app = app

    def __call__(self, environ, start_response, *args, **kwargs):
        wrapped_app_response = self.wrapped_app(environ, start_response)
        return [data[::-1] for data in wrapped_app_response]
```

refer: [`__call__`](https://www.geeksforgeeks.org/__call__-in-python/#:~:text=The%20__call__%20method,is%20a%20shorthand%20for%20x.)

Two things to note here:

1. First, web servers will talk to this middleware first and thus it must adhere to the same WSGI standards. That is, it should be a callable object that receives two params (`environ` and `start_response`) and then returns the response as an iterable.
2. Second, this middleware is getting the response from the app that it wraps and then it is tweaking it a little -- reversing the response, in this case. This is generally what middlewares are used for: tweaking the request and the response. We will see a much more useful example later in this course.

Now, if we insert this code in the example above, the full code will look like this:

```
# wsgi_demo.py

from wsgiref.simple_server import make_server


class Reverseware:
    def __init__(self, app):
        self.wrapped_app = app

    def __call__(self, environ, start_response, *args, **kwargs):
        wrapped_app_response = self.wrapped_app(environ, start_response)
        return [data[::-1] for data in wrapped_app_response]


def application(environ, start_response):
    response_body = [
        f'{key}: {value}' for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)

    status = '200 OK'

    response_headers = [
        ('Content-type', 'text/plain'),
    ]

    start_response(status, response_headers)

    return [response_body.encode('utf-8')]


server = make_server('localhost', 8000, app=Reverseware(application))
server.serve_forever()
```

Now, if you run it, you will see something like this in your browser:

![Reverse headers](https://s3.amazonaws.com/rahmonov.me/post-images/what-is-wsgi-anyway/wsgi-reverse-demo.png)

Beautiful!

## Conclusion

In this chapter, you learned what WSGI is and why it is needed. You also built a small, dummy framework (if you can even call it that) that simply receives any request and returns its environment variables in reverse as a response.

Alright, that's it for this chapter. If you want to learn more about WSGI, please see the updated [PEP 3333](https://www.python.org/dev/peps/pep-3333/).

See you in chapter two. Adios!

# Requests and Routing

##### Chapter 2

In the second chapter of the course, we will build the most important parts of the framework:

1. The request handlers (think Django views)
2. Routing -- both simple (like `/books/`) and parameterized (like `/books/{id}/`)

Now, before I start doing something new, I like to think about the end result. In this case, at the end of the day, we want to be able to use this framework in production and thus we want our framework to be served by a fast, lightweight, production-level application server. I have been using [Gunicorn](https://gunicorn.org/) in all of my projects the past few years, and I am very satisfied with the results. So, we'll go with Gunicorn as well.

Gunicorn is a WSGI HTTP Server, so it expects a specific entrypoint to our application, just like you learned in the last chapter.

To recap, to be WSGI-compatible, you need a callable object (a function or a class) that expects two parameters (`environ` and `start_response`) and returns an iterable response.

With those details out of the way, let's get started with our awesome framework.

## Requests

First, think of a name for your framework and create a folder with that name.

> We'll be using the name `bumbo` throughout this course. You can use that same name or come up with something new.

```
$ mkdir bumbo
```

Change directories to that folder. Create a virtual environment and activate it:

```
$ cd bumbo
$ python3.7 -m venv venv
$ source venv/bin/activate
```

Now, create a file named *app.py* where we will store our entrypoint for Gunicorn:

```
(venv)$ touch app.py
```

Inside *app.py*, add the following WSGI-compatible dummy function to see if it works with Gunicorn:

```python
# app.py
def app(environ, start_response):
    response_body = b"Hello, World!"
    status = "200 OK"
    start_response(status, headers=[])
    return iter([response_body])
```

It's nearly the same as the function you created in the previous chapter but much simpler. Instead of listing all the environment variables, it simply says "Hello, World!".

Now, to test, first install Gunicorn:

```
(venv)$ pip install gunicorn
```

Then, run the code:

```
(venv)$ gunicorn app:app
```

In the above command, the first `app` (left of the colon) is the file which you created and the second `app` (right of the colon) is the name of the function you just wrote. If all is good, you will see the following output in your console:

```
[2019-10-09 02:38:18 -0600] [24662] [INFO] Starting gunicorn 19.9.0
[2019-10-09 02:38:18 -0600] [24662] [INFO] Listening at: http://127.0.0.1:8000 (24662)
[2019-10-09 02:38:18 -0600] [24662] [INFO] Using worker: sync
[2019-10-09 02:38:18 -0600] [24665] [INFO] Booting worker with pid: 24665
```

If you see this, open your browser and go to [http://localhost:8000](http://localhost:8000/). You should see our good old friend: the `Hello, World!` message. Awesome! You'll build the rest of the framework off of this function.

Now, you will need quite a few helper methods, so let's turn this function into a class since they are much easier to write inside a class:

Create an *api.py* file:

```shell
(venv)$ touch api.py
```

Inside this file, create the following `API` class:

```python
# api.py
class API:
    def __call__(self, environ, start_response):
        response_body = b"Hello, World!"
        status = "200 OK"
        start_response(status, headers=[])
        return iter([response_body])
```

We'll look at what this class does together in a bit.

But first, delete everything inside *app.py* and write the following:

```python
# app.py
from api import API
app = API()
```

Restart (or re-run) Gunicorn and check the result in the browser. It should be the same as before since we converted our function named `app` to a class called `API` and overrode its `__call__` method.

It's worth noting that the `__call__` method is called when you call the instances of this class:

```python
app = API()
app()   #  this is where __call__ could be called
```

So, why are we only creating an instance of the `API` but not calling it in *app.py*? Simple: The calling of the instance of the `API` class is the responsibility of the web server (e.g., Gunicorn). In other words, it will be called when you run `gunicorn app:app`.

Now that you created the class, let's make the code look a bit more elegant because all those bytes (i.e., `b"Hello World"`) and the `start_response` function seem a bit confusing, right?

Thankfully, there is a cool package called [WebOb](https://docs.pylonsproject.org/projects/webob/en/stable/index.html) that provides classes for HTTP requests and responses by wrapping the `WSGI` request environment and response status, headers, and body. By using this package, we can pass the `environ` and `start_response` to the classes, which are provided by this package, and not have to deal with them ourselves.

> Before we continue, take a few minutes to look over the WebOb [documentation](https://docs.pylonsproject.org/projects/webob/en/stable/index.html). Make sure you understand how the [Request](https://docs.pylonsproject.org/projects/webob/en/stable/reference.html#request) and [Response](https://docs.pylonsproject.org/projects/webob/en/stable/reference.html#response) classes work. Briefly review the rest of the API as well.

Let's refactor this code!

### Refactoring

First, install WebOb:

```
(venv)$ pip install webob
```

Import the `Request` and `Response` classes at the beginning of the *api.py* file:

```
# api.py

from webob import Request, Response

...
```

And now you can use them inside the `__call__` method:

```python
# api.py
from webob import Request, Response

class API:
    def __call__(self, environ, start_response):
        request = Request(environ)

        response = Response()
        response.text = "Hello, World!"

        return response(environ, start_response)
```

Looks much better!

Restart Gunicorn and you should see the same result as before. The best part here is that the code above requires little explanation since it's mostly self-explanatory. You created a `request` and a `response` and then returned that `response`. Awesome!

Did you notice that we are not yet using the `request`? Let's change that. We can use it to get the user agent info from it. At the same time, we can refactor the `response` creation into its own method called `handle_request`. You will see why this is better later.

```python
# api.py

from webob import Request, Response


class API:
    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def handle_request(self, request):
        user_agent = request.environ.get("HTTP_USER_AGENT", "No User Agent Found")

        response = Response()
        response.text = f"Hello, my friend with this user agent: {user_agent}"

        return response
```

Restart Gunicorn and you should see the new message in the browser.

For example:

```shell
Hello, my friend with this user agent:
Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
```

Did you see it? Cool. Let's move on.

## Routing

At this point, your framework handles all the requests in the same way: Whatever request it receives, it simply returns the same response which is created in the `handle_request` method. Ultimately, it needs to be dynamic though. That is, it needs to serve the request coming from `/home/` differently than the one coming from `/about/`.

### Simple

To that end, inside *app.py*, create two methods that will handle those two requests:

```python
# app.py

from api import API


app = API()


def home(request, response):
    response.text = "Hello from the HOME page"


def about(request, response):
    response.text = "Hello from the ABOUT page"
```

Now, you need to somehow associate the two methods with the above mentioned paths -- `/home/` and `/about/`. I like the Flask way of managing this with decorators, which looks something like this:

```python
# app.py

from api import API


app = API()


@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"


@app.route("/about")
def about(request, response):
    response.text = "Hello from the ABOUT page"
```

Make the above changes to *app.py*.

So, what do you think? Look good? Let's create and implement the decorators!

As you can see, the `route` method above is a decorator that accepts a path and wraps the methods. Implement it in the `API` class like so:

```python
# api.py

from webob import Request, Response


class API:
    def __init__(self):
        # routes is a dict, keyi is path, value is a function
        self.routes = {}

    def __call__(self, environ, start_response):
        ...

    def route(self, path):
        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def handle_request(self, request):
        ...
```

What's happening?

First, In the `__init__` method, you defined a `dict` called `self.routes` where the framework will store paths as keys and handlers as values.

That `dict` will look something like this:

```
{
    "/home": <function home at 0x1100a70c8>,
    "/about": <function about at 0x1101a80c3>
}
```

Then, in the `route` method, you took a path as an argument and in the wrapper method you added this path in the `self.routes` dictionary as a key and the handler as a value.

At this point, you have all the pieces of the puzzle. You have the handlers and the paths associated with them. Now, when a request comes in, you need to check its `path`, find an appropriate handler, call that handler, and return an appropriate response.

Refactor the `handle_request` method to do just that:

```python
# api.py

from webob import Request, Response


class API:
    ...

    def handle_request(self, request):
        response = Response()

        for path, handler in self.routes.items():
            if path == request.path:
                handler(request, response)
                return response
```

Here, you iterated over `self.routes` and compared paths with the path of the request. If there is a match, you then called the handler associated with that path.

You should now have:

```python
# api.py

from webob import Request, Response


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def route(self, path):
        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def handle_request(self, request):
        response = Response()

        for path, handler in self.routes.items():
            if path == request.path:
                handler(request, response)
                return response
```

Restart Gunicorn again and try those paths in the browser:

1. http://localhost:8000/home
2. http://localhost:8000/about

You should see the corresponding messages. Pretty cool, right?

However, what happens if the path is not found?

Try http://localhost:8000/nonexistent/. You should see a big, ugly error message: `Internal Server Error`. If you turn to your console, you should see the issue:

```
TypeError: 'NoneType' object is not callable
```

You should see the same `TypeError` for the request for the favicon:

```
[2019-10-09 03:37:19 -0600] [28562] [ERROR] Error handling request /favicon.ico
```

To solve this, create a method that returns a simple HTTP response of "Not found.", with a 404 status code:

```python
# api.py

from webob import Request, Response


class API:
    ...

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    ...
```

Now, use it in the `handle_request` method:

```python
# api.py

from webob import Request, Response


class API:
    ...

    def handle_request(self, request):
        response = Response()

        for path, handler in self.routes.items():
            if path == request.path:
                handler(request, response)
                return response

        self.default_response(response)
        return response
```

Restart Gunicorn and try some nonexistent routes. You should see a lovely "Not found." page instead of the previous, unhandled error.

Next, refactor out finding a handler to its own method for the sake of readability:

```python
# api.py

from webob import Request, Response


class API:
    ...

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            if path == request_path:
                return handler

    ...
```

Just like before, it iterates over `self.route`, comparing paths with the request path, and returns the handler if the paths are the same or `None` if no handler is found.

Now, you can use it in your `handle_request` method:

```python
# api.py

from webob import Request, Response


class API:
    ...

    def handle_request(self, request):
        response = Response()

        handler = self.find_handler(request_path=request.path)

        if handler is not None:
            handler(request, response)
        else:
            self.default_response(response)

        return response
```

Much better, right? Fairly self-explanatory too.

Restart Gunicorn to ensure that everything works just like before.

### Parameterized

At this point, you have routes and handlers. It works, but the routes are simple. They don't support keyword parameters in the URL path. What if we wanted to have a route of `@app.route("/hello/{person_name}")` and be able to use `person_name` inside the handlers?

For example:

```python
def say_hello(request, response, person_name):
    resp.text = f"Hello, {person_name}"
```

For that, if someone goes to the `/hello/Matthew/`, the framework needs to be able to match this path with the registered `/hello/{person_name}/` and find the appropriate handler. Thankfully, there is a package called [Parse](https://github.com/r1chardj0n3s/parse) that does exactly that for you. Go ahead and install it:

```
(venv)$ pip install parse
```

Test this package out in the Python interpreter. First, open the Python interpreter by running `python` within your virtual environment, and then test out Parse like so:

```shell
>>> from parse import parse
>>> result = parse("Hello, {name}", "Hello, Matthew")
>>> print(result.named)
{'name': 'Matthew'}
```

As you can see, it parsed the string `Hello, Matthew` and was able to identify that `Matthew` corresponds to the provided `{name}`.

Now, use it in your `find_handler` method to find not only the method that corresponds to the path but also the keyword params that were provided. Make sure that you import it first:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    ...

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    ...
```

The `find_handler` method still iterates over `self.routes`; but, now instead of comparing the path to the request path, it tries to parse it and if there is a result, it returns both the handler and keyword params as a dictionary.

Now, you can use this inside `handle_request` to send params to the handlers:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    ...

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response
```

So, you are now getting both `handler` and `kwargs` from `self.find_handler` and passing `kwargs` to the handler via `**kwargs`.

You should now have:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def route(self, path):
        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response
```

Write a handler with this type of route and try it out in *app.py*:

```python
# app.py

from api import API


app = API()

...

@app.route("/hello/{name}")
def greeting(request, response, name):
    response.text = f"Hello, {name}"
```

Full file:

```python
# app.py

from api import API


app = API()


@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"


@app.route("/about")
def about(request, response):
    response.text = "Hello from the ABOUT page"

@app.route("/hello/{name}")
def greeting(request, response, name):
    response.text = f"Hello, {name}"
```

Restart Gunicorn and navigate to http://localhost:8000/hello/Matthew. You should see the wonderful message of `Hello, Matthew`. Try http://localhost:8000/hello/Mike and you should see `Hello, Mike`. Awesome, right?

Add a couple more such handlers on your own.

You can also indicate the [type](https://github.com/r1chardj0n3s/parse#format-specification) of the given params.

For example, you can define a digit type to an `age` param:

```python
@app.route("/tell/{age:d}")
```

Another example:

```python
@app.route("/sum/{num_1:d}/{num_2:d}")
def sum(request, response, num_1, num_2):
    total = int(num_1) + int(num_2)
    response.text = f"{num_1} + {num_2} = {total}"
```

> What happens if you pass in a non-digit? It will not be able to parse it and our `default_response` will do its job. Try this out.

## Conclusion

In this chapter, you started writing your own framework and built the most important features:

1. Request handlers
2. Simple and parameterized routing

You also learned about and how to use the WebOb and the third-party package named Parse.

We'll look at handling duplicate routes in the next chapter. See you there!

> Have you figured out why we refactored out the `response` creation into its own method called `handle_request` yet?

# Duplicate Routes and Class Based Handlers

##### Chapter 3

## Recap

To recap, in the first two chapters, you started writing your own Python-based web framework with the following features:

1. WSGI compatibility
2. Request Handlers
3. Routing -- both simple (`/books/`) and parameterized (`/books/{id}/`)

Take a few minutes to step back and reflect on what you've learned. Ask yourself the following questions:

1. What were my objectives?
2. How far did I get? How much is left?
3. How was the process? Am I on the right track?

Objectives:

1. Explain what WSGI is and why it's needed
2. Build a basic web framework and run it with Gunicorn, a WSGI-compatible server
3. Develop the core request handlers and routes

------

Moving right along, in this chapter, you'll add the following features to your framework:

- Check for duplicate routes
- Class-based handlers

Ready? Let's get started.

## Duplicate routes

Currently, you can add the same route any number of times to your framework without complaint.

For example:

```python
@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"


@app.route("/home")
def home2(request, response):
    response.text = "Hello from the SECOND HOME page"
```

The framework will not complain, but since a Python dictionary is used to store the routes, only the second route, `home2`, will work if you navigate to http://localhost:8000/home.

That's not good.

You want to make sure that the framework throws an exception if the user tries to add a duplicate route handler. Fortunately, since we're using a Python dict to store routes, we can check if the given path already exists in the dictionary. If it does, we can throw an exception. If it does not, we add the route per usual.

Before writing any code, take a moment to review the main `API` class:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def route(self, path):
        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response
```

Before moving on to the implementation, think on your own about how and where to handle the duplicate route check.

### Implementation

You need to change the `route` method so that it throws an exception if an existing route is being added again:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    ...

    def route(self, path):
        if path in self.routes:
            raise AssertionError("Such route already exists.")

        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    ...
```

Now, try adding the same route twice in *app.py*:

```python
# app.py

from api import API


app = API()


@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"

@app.route("/home")
def home2(request, response):
    response.text = "Hello from the SECOND HOME page"

...
```

Run (or restart) Gunicorn:

```
(venv)$ gunicorn app:app
```

You should see the following exception thrown:

```
AssertionError: Such route already exists.
```

Next, let's refactor the code to decrease that check to a single line:

```python
def route(self, path):
    assert path not in self.routes, "Such route already exists."

    def wrapper(handler):
        self.routes[path] = handler
        return handler

    return wrapper
```

You should now have:

```python
# api.py

from parse import parse
from webob import Request, Response


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def route(self, path):
        assert path not in self.routes, "Such route already exists."

        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response
```

Voilà! Onto the next feature.

## Class Based Handlers

With the function-based handlers complete, let's add class-based ones. Much like Django's [class-based views](https://docs.djangoproject.com/en/2.2/topics/class-based-views/), these are more suitable for larger, more complicated handlers.

End goal:

```python
@app.route("/book")
class BooksResource:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"
```

Now the dict that stores routes, `self.routes`, can contain both classes and functions as values. Thus, when we find a handler in the `handle_request()` method, we need to check if the handler is a function or if it is a class.

- If it's a function, it should work as it does now.
- If it's a class, depending on the request method, you should call the appropriate method of the class. That is, if the request method is `GET` you should call the `get()` method of the class, if it is `POST` you should call the `post` method, and so on.

Here's how the `handle_request()` method currently looks:

```python
def handle_request(self, request):
    response = Response()

    handler, kwargs = self.find_handler(request_path=request.path)

    if handler is not None:
        handler(request, response, **kwargs)
    else:
        self.default_response(response)

    return response
```

To update this to handle classes, the first thing we need to do is check if the found handler is a class. For that, we can use the built-in `inspect` module like this:

```python
def handle_request(self, request):
    response = Response()

    handler, kwargs = self.find_handler(request_path=request.path)

    if handler is not None:
        if inspect.isclass(handler):
            pass   # class-based handler is being used
        else:
            handler(request, response, **kwargs)
    else:
        self.default_response(response)

    return response
```

Make sure you add the import:

```python
import inspect
```

Now, if a class-based handler is being used, we need to find the appropriate method of the class based on the given request method. For that we can use the built-in `getattr` function:

```python
def handle_request(self, request):
    response = Response()

    handler, kwargs = self.find_handler(request_path=request.path)

    if handler is not None:
        if inspect.isclass(handler):
            handler_function = getattr(handler(), request.method.lower(), None)
            pass
        else:
            handler(request, response, **kwargs)
    else:
        self.default_response(response)

    return response
```

`getattr` accepts an object instance as the first param and the attribute name to get as the second. The third argument is the value to return if nothing is found.

So, `GET` will return `get`, `POST` will return `post`, and `some_other_attribute` will return `None`. If the `handler_function` is `None`, it means that such function was not implemented in the class and the request method is not allowed:

```python
if inspect.isclass(handler):
    handler_function = getattr(handler(), request.method.lower(), None)
    if handler_function is None:
        raise AttributeError("Method not allowed", request.method)
```

If the `handler_function` was actually found, then you call it:

```python
if inspect.isclass(handler):
    handler_function = getattr(handler(), request.method.lower(), None)
    if handler_function is None:
        raise AttributeError("Method not allowed", request.method)
    handler_function(request, response, **kwargs)
```

Now the whole method looks like this:

```python
def handle_request(self, request):
    response = Response()

    handler, kwargs = self.find_handler(request_path=request.path)

    if handler is not None:
        if inspect.isclass(handler):
            handler_function = getattr(handler(), request.method.lower(), None)
            if handler_function is None:
                raise AttributeError("Method not allowed", request.method)
            handler_function(request, response, **kwargs)
        else:
            handler(request, response, **kwargs)
    else:
        self.default_response(response)

    return response
```

Rather than having a `handler_function` and a `handler`, we can refactor the code to make it more elegant:

```python
def handle_request(self, request):
    response = Response()

    handler, kwargs = self.find_handler(request_path=request.path)

    if handler is not None:
        if inspect.isclass(handler):
            handler = getattr(handler(), request.method.lower(), None)
            if handler is None:
                raise AttributeError("Method not allowed", request.method)

        handler(request, response, **kwargs)
    else:
        self.default_response(response)

    return response
```

And that's it. You can now test support for class-based handlers.

For reference, *api.py* should now look like this:

```python
# api.py

import inspect

from parse import parse
from webob import Request, Response


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def route(self, path):
        assert path not in self.routes, "Such route already exists."

        def wrapper(handler):
            self.routes[path] = handler
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            if inspect.isclass(handler):
                handler = getattr(handler(), request.method.lower(), None)
                if handler is None:
                    raise AttributeError("Method not allowed", request.method)

            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response
```

### Sanity Check

If you haven't already, add the following handler to *app.py*:

```python
# app.py

from api import API


app = API()

...

@app.route("/book")
class BooksResource:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"
```

Now, restart Gunicorn and navigate to http://localhost:8000/book. You should see the message `Books Page`. Test the `POST` method as well by sending the following request with the help of `curl` in the console:

```
$ curl -X POST http://localhost:8000/book
```

You should see the corresponding message of `Endpoint to create a book`. And there you go. You have added support for class-based handlers. Play around them for a bit by implementing other methods such as `put` and `delete`. Make sure to test a nonexistent path as well.

See you in the next chapter.

# Unit Tests and Test Client

##### Chapter 4

In this chapter, we'll pause feature development for a bit and write some unit tests with Pytest. We'll also add a test client so we can start testing the endpoints.

## Unit Tests

Start by installing [Pytest](https://docs.pytest.org/):

```shell
(venv)$ pip install pytest
```

> New to Pytest? Be sure to review the [Getting Started](https://docs.pytest.org/en/latest/getting-started.html) guide. Try writing a few unit tests on your own before moving on.

Create a file where you'll write the tests:

```shell
(venv)$ touch test_bumbo.py
```

Next, create a [fixture](https://docs.pytest.org/en/latest/fixture.html) for your `API` class that you can use in every test:

```python
# test_bumbo.py

import pytest

from api import API


@pytest.fixture
def api():
    return API()
```

Now, for your first unit test, start with something simple: test if a route can be added. If it doesn't throw an exception, it means that the test passes successfully:

```python
# test_bumbo.py

...

def test_basic_route_adding(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"
```

Make sure you have activated your virtualenv with `source venv/bin/activate` and run the tests like so:

```
(venv)$ pytest test_bumbo.py
```

You should see something like the following:

```
======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
collected 1 item

test_bumbo.py .                                                                               [100%]

========================================= 1 passed in 0.20s =========================================
```

Now, test that it throws an exception if you try to add an existing route:

```python
# test_bumbo.py

...

def test_route_overlap_throws_exception(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"

    with pytest.raises(AssertionError):
        @api.route("/home")
        def home2(req, resp):
            resp.text = "YOLO"
```

Run the tests again. Both of them should pass.

What else should we test? How about the default response, parameterized routing, status codes. Anything else?

In order to test those, we'll need to send an HTTP request to the handlers and then test the response, so let's configure a test client to handle this.

## Test Client

By far the most popular way of sending HTTP requests in Python is the [Requests](https://github.com/psf/requests) library.

Now, since Requests only ships with a single [Transport Adapter](https://requests.readthedocs.io/en/master/user/advanced/#transport-adapters), the [HTTPAdapter](https://requests.readthedocs.io/en/master/api/#requests.adapters.HTTPAdapter), we'd have to fire up Gunicorn before each test run in order to use it in the unit tests. That defeats the purpose of unit tests, though: Unit tests should be self-sustained. Fortunately, we can use the [WSGI Transport Adapter for Requests](https://github.com/seanbrant/requests-wsgi-adapter) library to create a test client that will make the tests self-sustained.

Go ahead and install both of these wonderful libraries:

```
(venv)$ pip install requests requests-wsgi-adapter
```

Add the following method called `test_session` to the main `API` class in *api.py*:

```python
# api.py

import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter


class API:

    ...

    def test_session(self, base_url="http://testserver"):
        session = RequestsSession()
        session.mount(prefix=base_url, adapter=RequestsWSGIAdapter(self))
        return session
```

As written [here](https://requests.readthedocs.io/en/master/user/advanced/#transport-adapters), to use the Requests WSGI Adapter, you need to mount it to a Session object. That way, any request made using this `test_session` whose URL starts with the given prefix, will use the given `RequestsWSGIAdapter`.

> Note the `base_url` that you will be using in all your unit tests to send requests.

Great, now you can use this `test_session` to create a test client.

### Fixtures

It's a good idea to keep fixtures in a separate file. So, create a *conftest.py* file and move the `api` fixture to this file so that it looks like this:

```python
# conftest.py

import pytest

from api import API


@pytest.fixture
def api():
    return API()
```

This file is where Pytest looks for fixtures by default. Remember to delete this `api` fixture from *test_bumbo.py*.

Now, create the test client fixture:

```python
# conftest.py

...

@pytest.fixture
def client(api):
    return api.test_session()
```

The `client` uses the `api` fixture to return the `test_session` that you wrote earlier. Now you can use this `client` fixture in your unit tests.

To ensure this works -- e.g., the `client` can send a request -- add the following unit test to *test_bumbo.py*:

```python
def test_bumbo_test_client_can_send_requests(api, client):
    RESPONSE_TEXT = "THIS IS COOL"

    @api.route("/hey")
    def cool(req, resp):
        resp.text = RESPONSE_TEXT

    assert client.get("http://testserver/hey").text == RESPONSE_TEXT
```

Run the unit tests via `pytest test_bumbo.py`. And voila! You see that all the tests pass:

```
======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
collected 3 items

test_bumbo.py ...                                                                             [100%]

========================================= 3 passed in 0.08s =========================================
```

### More Tests

Add a couple more unit tests to test the most important parts.

Test that the URL parameters work as expected:

```python
def test_parameterized_route(api, client):
    @api.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
    assert client.get("http://testserver/ashley").text == "hey ashley"
```

Tests that if a request is sent to a non-existent route, a 404 (Not Found) response is returned:

```python
def test_default_404_response(client):
    response = client.get("http://testserver/doesnotexist")

    assert response.status_code == 404
    assert response.text == "Not found."
```

Add a few tests for the class based handlers:

```python
def test_class_based_handler_get(api, client):
    response_text = "this is a get request"

    @api.route("/book")
    class BookResource:
        def get(self, req, resp):
            resp.text = response_text

    assert client.get("http://testserver/book").text == response_text


def test_class_based_handler_post(api, client):
    response_text = "this is a post request"

    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = response_text

    assert client.post("http://testserver/book").text == response_text


def test_class_based_handler_not_allowed_method(api, client):
    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = "yolo"

    with pytest.raises(AttributeError):
        client.get("http://testserver/book")
```

The first two test the `get` and `post` methods, respectively. The third one tests that an exception is raised if a request is sent to a non-implemented method.

You should now have:

```python
# test_bumbo.py

import pytest

from api import API


def test_basic_route_adding(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"


def test_route_overlap_throws_exception(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"

    with pytest.raises(AssertionError):
        @api.route("/home")
        def home2(req, resp):
            resp.text = "YOLO"


def test_bumbo_test_client_can_send_requests(api, client):
    RESPONSE_TEXT = "THIS IS COOL"

    @api.route("/hey")
    def cool(req, resp):
        resp.text = RESPONSE_TEXT

    assert client.get("http://testserver/hey").text == RESPONSE_TEXT


def test_parameterized_route(api, client):
    @api.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
    assert client.get("http://testserver/ashley").text == "hey ashley"


def test_default_404_response(client):
    response = client.get("http://testserver/doesnotexist")

    assert response.status_code == 404
    assert response.text == "Not found."


def test_class_based_handler_get(api, client):
    response_text = "this is a get request"

    @api.route("/book")
    class BookResource:
        def get(self, req, resp):
            resp.text = response_text

    assert client.get("http://testserver/book").text == response_text


def test_class_based_handler_post(api, client):
    response_text = "this is a post request"

    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = response_text

    assert client.post("http://testserver/book").text == response_text


def test_class_based_handler_not_allowed_method(api, client):
    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = "yolo"

    with pytest.raises(AttributeError):
        client.get("http://testserver/book")
```

Ensure they pass:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
collected 8 items

test_bumbo.py ........                                                                        [100%]

========================================= 8 passed in 0.12s =========================================
```

### Test Coverage

At this point, the [test coverage](https://en.wikipedia.org/wiki/Code_coverage) should be be close to 100%. To verify, install the [pytest-cov](https://pytest-cov.readthedocs.io/) package that checks what test coverage looks like in the project:

```
(venv)$ pip install pytest-cov
```

Add a *.coveragerc* file, which is used to [configure the coverage tool](https://coverage.readthedocs.io/en/latest/config.html), to ignore the "venv" virtual environment directory along with the *test_bumbo.py*, *conftest.py*, and *app.py* files:

```
[run]
omit = venv/*,test_bumbo.py,conftest.py,app.py
```

This will help you get an accurate coverage percentage for the project.

Now run the tests with coverage:

```
(venv)$ pytest --cov=. test_bumbo.py
```

You should see something like:

```
======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 8 items

test_bumbo.py ........                                                                        [100%]

---------- coverage: platform darwin, python 3.7.4-final-0 -----------
Name     Stmts   Miss  Cover
----------------------------
api.py      42      0   100%


========================================= 8 passed in 0.11s =========================================
```

You should have 100% coverage on the most important file, *api.py*. Nice!

As we add more features to the framework, we'll be adding more unit tests as well so that the framework is more reliable, easier to refactor, and more fun to work with. We'll also take a test-driven approach where appropriate and write the tests first.

See you in the next chapter!

# Django-like Routes and Templates

##### Chapter 5

## Recap

To recap, in the previous chapters, you added the following features to your Python-based web framework:

1. WSGI compatibility
2. Request Handlers
3. Routing -- both simple (`/books/`) and parameterized (`/books/{id}/`)
4. Check for duplicate routes
5. Class-based handlers
6. Unit Tests
7. Test Client

Take a few minutes to step back and reflect on what you've learned. Ask yourself the following questions:

1. What were my objectives?
2. How far did I get? How much is left?
3. How was the process? Am I on the right track?

Objectives:

1. Explain what WSGI is and why it's needed
2. Build a basic web framework and run it with Gunicorn, a WSGI-compatible server
3. Develop the core request handlers and routes
4. Implement class-based route handlers
5. Test your framework with unit tests and practice test-driven development
6. Build a test client to test the API without having to spin up the server

------

In this chapter, you'll add support for Django-like routes and templates.

## Django-like Routes

Let's quickly look at an alternative way to add routes.

Right now, routes are added like so:

```python
@app.route("/home")
def handler(req, resp):
    resp.text = "YOLO"
```

That is, routes are added as decorators, which is similar to Flask. Some people may like the Django way of registering URLs. So, why don't you give them a choice to add routes like this:

```python
def handler(req, resp):
    resp.text = "YOLO"


def handler2(req, resp):
    resp.text = "YOLO2"


app.add_route("/home", handler)
app.add_route("/about", handler2)
```

The `add_route` method should do two things:

1. Check if the route is already registered or not
2. Register it

Go ahead and add a unit test for it:

```python
# test_bumbo.py

import pytest

from api import API

...

def test_alternative_route(api, client):
    response_text = "Alternative way to add a route"

    def home(req, resp):
        resp.text = response_text

    api.add_route("/alternative", home)

    assert client.get("http://testserver/alternative").text == response_text
```

Run the tests:

```shell
(venv)$ pytest test_bumbo.py
```

It should fail:

```shell
========================================================= test session starts =========================================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 9 items

test_bumbo.py ........F                                                                                                         [100%]

============================================================== FAILURES ===============================================================
_______________________________________________________ test_alternative_route ________________________________________________________

api = <api.API object at 0x106ebe550>, client = <requests.sessions.Session object at 0x106ebea10>

    def test_alternative_route(api, client):
        response_text = "Alternative way to add a route"

        def home(req, resp):
            resp.text = response_text

>       api.add_route("/alternative", home)
E       AttributeError: 'API' object has no attribute 'add_route'

test_bumbo.py:90: AttributeError
===================================================== 1 failed, 8 passed in 0.10s =====================================================
```

Implement the method in the `API` class:

```python
# api.py

import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter


class API:
    ...

    def add_route(self, path, handler):
        assert path not in self.routes, "Such route already exists."

        self.routes[path] = handler

    ...
```

Does this code look familiar to you? It should. You already wrote the exact same code in the `route` decorator:

```python
def route(self, path):
    assert path not in self.routes, "Such route already exists."

    def wrapper(handler):
        self.routes[path] = handler
        return handler

    return wrapper
```

To follow the [DRY principle](https://en.wikipedia.org/wiki/Don't_repeat_yourself), we can use `add_route` inside the `route` decorator like so:

```python
# api.py

import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter


class API:
    ...

    def add_route(self, path, handler):
        assert path not in self.routes, "Such route already exists."

        self.routes[path] = handler

    def route(self, path):
        def wrapper(handler):
            self.add_route(path, handler)
            return handler

        return wrapper

    ...
```

Run the tests. You should see them pass, which means that the new feature is working and the old ones were not broken.

```
========================================================= test session starts =========================================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 9 items

test_bumbo.py .........                                                                                                         [100%]

========================================================== 9 passed in 0.06s ==========================================================
```

Your *api.py* file should now look like this:

```python
# api.py

import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter


class API:
    def __init__(self):
        self.routes = {}

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def add_route(self, path, handler):
        assert path not in self.routes, "Such route already exists."

        self.routes[path] = handler

    def route(self, path):
        def wrapper(handler):
            self.add_route(path, handler)
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        if handler is not None:
            if inspect.isclass(handler):
                handler = getattr(handler(), request.method.lower(), None)
                if handler is None:
                    raise AttributeError("Method not allowed", request.method)

            handler(request, response, **kwargs)
        else:
            self.default_response(response)

        return response

    def test_session(self, base_url="http://testserver"):
        session = RequestsSession()
        session.mount(prefix=base_url, adapter=RequestsWSGIAdapter(self))
        return session
```

Finally, add a new handler to *app.py*:

```python
def handler(req, resp):
    resp.text = "sample"

app.add_route("/sample", handler)
```

Restart Gunicorn and ensure http://localhost:8000/sample works as expected.

Onto the next feature!

## Templates

When implementing something new, I like to do something called "README-driven development". It's a technique where you write down what you want your API to look like before implementing it. Let's try this for the next feature.

Say you have this template that you want to use in the handler:

```html
<html>
    <header>
        <title>{{ title }}</title>
    </header>

    <body>
        The name of the framework is {{ name }}
    </body>

</html>
```

`{{ title }}` and `{{ name }}` are variables that are sent from a request handler that look like:

```python
app = API(templates_dir="templates")


@app.route("/home")
def handler(req, resp):
    resp.body = app.template(
        "home.html",
        context={"title": "Awesome Framework", "name": "Bumbo"}
    )
```

To keep it as simple as possible, let's use a single method, `app.template`, that takes a template name and context as params and renders that template with the given params. Also, the templates directory will be configurable just like above.

First, write a unit test for the planned `template` method:

```python
# test_bumbo.py

import pytest

from api import API

...

def test_template(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.body = api.template("index.html", context={"title": "Some Title", "name": "Some Name"}).encode()

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Some Title" in response.text
    assert "Some Name" in response.text
```

In this unit test, we are ensuring that the variables sent as a context are being rendered in the template and also that the content type is `text/html`. Ignore the `.encode()` called after `api.template()` for now. I will explain why this is needed later. Run the tests. It should fail.

With the API designed and test written, you can now implement it.

### Jinja2

For templates support, I think that [Jinja2](http://jinja.pocoo.org/) is the best choice. It's a modern, designer-friendly templating language for Python, modelled after Django's templates. So, if you know Django it should feel right at home.

Jinja2 uses a central object called the template `Environment`. You will configure this environment upon application initialization and load templates with the help of this environment.

Here's an example of how to create and configure a new `Environment`:

```python
import os
from jinja2 import Environment, FileSystemLoader


templates_env = Environment(loader=FileSystemLoader(os.path.abspath("templates")))
```

`FileSystemLoader` loads templates from the file system. This loader can find templates in folders on the file system and is the preferred way to load them. It takes the path to the templates directory as a parameter.

We can then use `templates_env` like so:

```python
templates_env.get_template("index.html").render({"title": "Awesome Framework", "name": "Bumbo"})
```

With that, let's add Jinja2 to the framework.

Start by installing it:

```shell
(venv)$ pip install jinja2
```

Then, create the `Environment` object in the `__init__` method of the `API` class after the necessary imports:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    def __init__(self, templates_dir="templates"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

    ...
```

When compared to the example above, we did just about the same thing except that we gave `templates_dir` a default value of `templates` so that users don't have to write it if they don't want to.

Now you have everything you need to implement the `template` method:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    ...

    def template(self, template_name, context=None):
        if context is None:
            context = {}

        return self.templates_env.get_template(template_name).render(**context)
```

This should be fairly straightforward, right? The only thing you may be wondering about is why I gave `context` a default value of `None`, checked if it is `None`, and then set the value to an empty dictionary, `{}`? Why not just give it a default value of `{}` in the declaration? Well, `dict` is a mutable object and it is a bad practice to set a mutable object as a default value in Python. You can read more about this [here](https://docs.python-guide.org/writing/gotchas/#mutable-default-arguments). This is an excellent, and frequently asked, interview question for Pythonistas.

Now, create the `templates` folder:

```shell
(venv)$ mkdir templates
```

Next, create an *index.html* file by running `touch templates/index.html`, and then add the following HTML:

```html
<html>
  <header>
    <title>{{ title }}</title>
  </header>

  <body>
    <h1>The name of the framework is {{ name }}</h1>
  </body>
</html>
```

At this point, you have everything ready. Go ahead and run the unit tests again. They should pass.

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 10 items

test_bumbo.py ..........                                                                      [100%]

======================================== 10 passed in 0.08s =========================================
```

You can now go ahead and create a handler in *app.py*:

```python
# app.py

from api import API


app = API()

...

@app.route("/template")
def template_handler(req, resp):
    resp.body = app.template(
        "index.html", context={"name": "Bumbo", "title": "Best Framework"})

...
```

That's it. Well, almost. Start Gunicorn and navigate to http://localhost:8000/template. You should see a big bold `Internal Server Error` message. If you turn to the console, you should see the following error:

```
TypeError: You cannot set Response.body to a text object (use Response.text)
```

Why?

The `resp.body` expects bytes while the `template` method returns a unicode string.

Thus, we need to encode it:

```python
@app.route("/template")
def template_handler(req, resp):
    resp.body = app.template(
        "index.html",
        context={"name": "Bumbo", "title": "Best Framework"}
    ).encode()
```

Restart Gunicorn, and test the route again. You should see your template in all its glory. In a future chapter, you will remove the need to `encode` and make your API prettier.

In the coming chapters, you'll add a support for static files, custom exception handlers, and middleware.

See you in the next chapter!

# Exception Handlers and Static Files

##### Chapter 6

In this chapter, we'll add the following features to the framework:

1. Custom exception handlers
2. Support for static files

## Custom Exception Handlers

Exceptions inevitably happen. Users may do something that we don't expect. We all write some code that doesn't work on some occasions. Users may go to a non-existent page. With what you have right now, if some exception happens, an ugly `Internal Server Error` message is shown. Instead, a better one should be shown. Something along the lines of `Oops! Something went wrong.` or `Please, contact our customer support`. We need to be able to catch those exceptions and handle them in a better way.

The end goal will be to have an exception handler like this:

```python
def custom_exception_handler(request, response, exception_cls):
    response.text = str(exception_cls)

app.add_exception_handler(custom_exception_handler)
```

Here a custom exception handler is created. It looks almost like a simple request handler, except that it has `exception_cls` as its third argument. Now, if you have a request handler that throws an exception, this custom exception handler should be called.

Sound good?

Let's start with a test:

```python
# test_bumbo.py

import pytest

from api import API

...

def test_custom_exception_handler(api, client):
    def on_exception(req, resp, exc):
        resp.text = "AttributeErrorHappened"

    api.add_exception_handler(on_exception)

    @api.route("/")
    def index(req, resp):
        raise AttributeError()

    response = client.get("http://testserver/")

    assert response.text == "AttributeErrorHappened"
```

In this unit test, we are asserting that when an `AttributeError` is raised in a handler, it is caught by the custom exception handler and its text is changed.

Run the tests to ensure it fails:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 11 items

test_bumbo.py ..........F                                                                     [100%]

============================================= FAILURES ==============================================
___________________________________ test_custom_exception_handler ___________________________________

api = <api.API object at 0x108bd2c90>, client = <requests.sessions.Session object at 0x108bd2ad0>

    def test_custom_exception_handler(api, client):
        def on_exception(req, resp, exc):
            resp.text = "AttributeErrorHappened"

>       api.add_exception_handler(on_exception)
E       AttributeError: 'API' object has no attribute 'add_exception_handler'

test_bumbo.py:111: AttributeError
=================================== 1 failed, 10 passed in 0.23s ====================================
```

Time to implement the feature.

The first thing we need is to define a variable inside the main API class where we can store the exception handler:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    def __init__(self, templates_dir="templates"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

    ...
```

Next, add the `add_exception_handler` method:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    ...

    def add_exception_handler(self, exception_handler):
        self.exception_handler = exception_handler
```

Having registered the custom exception handler, we need to call it when an exception happens. Where do exceptions happen? That's right: When handlers are called. Since we call the handlers inside the `handle_request` method, we need to wrap it with a `try/except` clause and call the custom exception handler in the `except` part. We also need to make sure that if no exception handler has been registered, the exception is propagated. This is done by checking if an exception handler is registered; and, if not, raising the exception in the `except` clause.

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    ...

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        try:
            if handler is not None:
                if inspect.isclass(handler):
                    handler = getattr(handler(), request.method.lower(), None)
                    if handler is None:
                        raise AttributeError("Method not allowed", request.method)

                handler(request, response, **kwargs)
            else:
                self.default_response(response)
        except Exception as e:
            if self.exception_handler is None:
                raise e
            else:
                self.exception_handler(request, response, e)

        return response

    ...
```

That's it. Try running the tests again. They should all pass.

```
======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 11 items

test_bumbo.py ...........                                                                     [100%]

======================================== 11 passed in 0.07s =========================================
```

Great! Our new feature works!

Want to see this in action?

First, add the exception handler to *app.py*:

```python
def custom_exception_handler(request, response, exception_cls):
    response.text = str(exception_cls)

app.add_exception_handler(custom_exception_handler)
```

Then, add another handler used for testing:

```python
# app.py

from api import API


app = API()

...

@app.route("/exception")
def exception_throwing_handler(request, response):
    raise AssertionError("This handler should not be used.")

...
```

Restart Gunicorn and navigate to http://localhost:8000/exception. You should see our little message instead of the ugly default one. You can also test with curl, by using the wrong request method:

```
$ curl -X PUT http://localhost:8000/book
```

You should see:

```
('Method not allowed', 'PUT')
```

If you want to take it one step further, create a nice template and use your `api.template()` method inside the exception handler.

If you want to take it even further, handle different kinds of exceptions in different ways. For example, you could handle 404 (Not Found) exceptions different from 500 (Server Error) exceptions.

Just keep in mind that the framework doesn't support static files yet, so you won't be able to add style with CSS or interactivity with JavaScript. Fortunately, that's exactly what we're doing next!

## Static Files

Templates are not truly templates without good CSS and JavaScript, so let's add a support for such files then!

But before you get started, add a unit test first:

```python
# test_bumbo.py

import pytest

from api import API

...

def test_404_is_returned_for_nonexistent_static_file(client):
    assert client.get(f"http://testserver/main.css)").status_code == 404
```

This one simply tests if a 404 (Not Found) response is returned if a request is sent for a nonexistent static file.

Next, add a unit test that checks if the correct static file is returned if it exists:

```python
# test_bumbo.py

import pytest

from api import API

FILE_DIR = "css"
FILE_NAME = "main.css"
FILE_CONTENTS = "body {background-color: red}"

# helpers

def _create_static(static_dir):
    asset = static_dir.mkdir(FILE_DIR).join(FILE_NAME)
    asset.write(FILE_CONTENTS)

    return asset


# tests

...

def test_assets_are_served(tmpdir_factory):
    static_dir = tmpdir_factory.mktemp("static")
    _create_static(static_dir)
    api = API(static_dir=str(static_dir))
    client = api.test_session()

    response = client.get(f"http://testserver/{FILE_DIR}/{FILE_NAME}")

    assert response.status_code == 200
    assert response.text == FILE_CONTENTS
```

This one is a little complicated:

1. Three constants are created for a file directory, name, and contents.
2. A helper method is created that creates a static file under the given folder.
3. In the actual test, we first created a folder for static files with the help of the `tmpdir_factory` fixture that is a Pytest built-in. This fixture creates a temporary folder on your system, which we used to temporarily store static files used in the tests.
4. The `_create_static` method is called to create a temporary static file (`main.css`).
5. We then created an instance of our API class with the `static_dir` we just created.
6. Finally, we sent a request for the newly created static file and asserted that the status code is 200 and its contents are correct.

Run the tests to ensure it fails.

### WhiteNoise

To get it to pass, start by installing [WhiteNoise](http://whitenoise.evans.io/) for static file serving:

```
(venv)$ pip install whitenoise
```

To configure WhiteNoise, we just need to wrap the WSGI app and give it the static folder path as a parameter. But, before we do that, recall what the `__call__` method currently looks like:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    ...

    def __call__(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    ...
```

This is basically the entry point into the WSGI app, which is exactly what we need to wrap with WhiteNoise. Let's refactor its content to a separate method so that it will be easier to wrap:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader


class API:
    ...

    def __call__(self, environ, start_response):
        return self.wsgi_app(environ, start_response)

    def wsgi_app(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    ...
```

At this point, nothing has changed, we just did some refactoring.

Next, import WhiteNoise and then, in the constructor, initialize a new instance of it:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise


class API:
    def __init__(self, templates_dir="templates", static_dir="static"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

        self.whitenoise = WhiteNoise(self.wsgi_app, root=static_dir)

    ...
```

As you can see, we wrapped `wsgi_app` with WhiteNoise and gave it a path to the static folder as the second param. Also note that we made`static_dir` configurable by making it an argument to the `__init__` method.

The only thing left to do is make `self.whitenoise` an entrypoint to the framework:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise


class API:
    ...

    def __call__(self, environ, start_response):
        return self.whitenoise(environ, start_response)

    ...
```

With everything in place, run the tests to ensure they pass:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 13 items

test_bumbo.py .............                                                                   [100%]

======================================== 13 passed in 0.10s =========================================
```

It's now time to test it manually in the browser.

Create a "static" folder in the project root and create a *main.css* file with the following content:

```css
body {
    background-color: chocolate;
}
```

Add the newly created CSS file to the *templates/index.html* template:

```html
<html>
  <header>
    <title>{{ title }}</title>

    <link href="/main.css" type="text/css" rel="stylesheet">
  </header>

  <body>
    <h1>The name of the framework is {{ name }}</h1>
  </body>
</html>
```

Restart Gunicorn and navigate to http://localhost:8000/template. The background should now be chocolate, not white, meaning that your static file is being served. Awesome!

See you in the next chapter!

# Middleware

##### Chapter 7

## Recap

To recap, in the previous chapters, you added the following features to your Python-based web framework:

1. WSGI compatibility
2. Request Handlers
3. Routing -- both simple (`/books/`) and parameterized (`/books/{id}/`)
4. Check for duplicate routes
5. Class-based handlers
6. Unit Tests
7. Test Client
8. Alternative way to add routes (like Django)
9. Support for templates
10. Custom exception handlers
11. Support for static files

Take a few minutes to step back and reflect on what you've learned. Ask yourself the following questions:

1. What were my objectives?
2. How far did I get? How much is left?
3. How was the process? Am I on the right track?

Objectives:

1. Explain what WSGI is and why it's needed
2. Build a basic web framework and run it with Gunicorn, a WSGI-compatible server
3. Develop the core request handlers, routes, and templates
4. Implement class-based route handlers
5. Test your framework with unit tests and practice test-driven development
6. Build a test client to test the API without having to spin up the server
7. Implement custom exception handlers to ensure 404 (Not Found) and 500 (Internal Server Error) errors are handled gracefully
8. Develop a solution for managing static files in the framework

------

In this chapter, you'll add support for middlewares.

## Middleware

Need a little recap of what middlewares are and how they work? Jump back to the [first chapter](https://testdriven.io/courses/python-web-framework/wsgi/). Otherwise, this part may seem a little confusing. You should have an understanding of what they are and how they work before moving on.

With that, let's look at what middlewares are used for.

Basically, middleware is a component that can modify an HTTP request and/or response and is designed to be chained together to form a pipeline of behavioral changes during request processing. Examples of middleware tasks are request logging and HTTP authentication. The main point is that neither of these are fully responsible for responding to a client. Instead, each middleware changes the behavior in some way as part of the pipeline, leaving the actual response to come from something later in the pipeline. In our case, that something which actually responds to a client is our request handlers. Middlewares are wrappers around our WSGI app that have the ability to modify requests and responses.

From the bird's eye view, the code will look like this:

```
FirstMiddleware(SecondMiddleware(our_wsgi_app))
```

So, when a request comes in, it hits `FirstMiddleware` first, which modifies the request in some way and sends it over to `SecondMiddleware`. `SecondMiddleware` then modifies the request and sends it over to `our_wsgi_app`. The app handles the request, prepares the response and sends it back to `SecondMiddleware`. It can modify the response if it wants before sending it back to `FirstMiddleware`. The response is modified again, and then `FirstMiddleware` sends it back to the web server (e.g., Gunicorn).

Let's implement two middleware methods that change the request and response -- `process_request` and `process_response`, respectively. We'll also create an `add_middleware` method for adding new middlewares to the pipeline:

```python
app = API()
app.add_middleware(SomeMiddleware)
```

## Test

With the API designed, add a new unit test:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware

...

def test_middleware_methods_are_called(api, client):
    process_request_called = False
    process_response_called = False

    class CallMiddlewareMethods(Middleware):
        def __init__(self, app):
            super().__init__(app)

        def process_request(self, req):
            nonlocal process_request_called
            process_request_called = True

        def process_response(self, req, resp):
            nonlocal process_response_called
            process_response_called = True

    api.add_middleware(CallMiddlewareMethods)

    @api.route('/')
    def index(req, res):
        res.text = "YOLO"

    client.get('http://testserver/')

    assert process_request_called is True
    assert process_response_called is True
```

Here, a middleware class is created and added to the pipeline. The test checks if methods of the middleware are called.

Run the tests to ensure it fails:

```shell
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 0 items / 1 errors

============================================== ERRORS ===============================================
__________________________________ ERROR collecting test_bumbo.py ___________________________________
ImportError while importing test module '/Users/michael.herman/repos/testdriven/python-web-framework/bumbo/test_bumbo.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
test_bumbo.py:6: in <module>
    from middleware import Middleware
E   ModuleNotFoundError: No module named 'middleware'
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 errors during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
========================================= 1 error in 0.16s ==========================================
```

## Implementation

We'll start by creating a `Middleware` class that other middlewares will inherit from, which also wraps our WSGI app:

```
(venv)$ touch middleware.py
```

Now, you can begin the `Middleware` class:

```python
# middleware.py


class Middleware:
    def __init__(self, app):
        self.app = app
```

As mentioned above, it should wrap the WSGI app and in case of multiple middlewares that `app` can also be another middleware.

As a base middleware class, it should also have the ability to add another middleware to the stack:

```python
# middleware.py

class Middleware:
    def __init__(self, app):
        self.app = app

    def add(self, middleware_cls):
        self.app = middleware_cls(self.app)
```

Here, we wrapped the given middleware class around the current app.

It should also have its main methods which are for request and response processing. For now, they will do nothing. The child classes that will inherit from this class will implement these methods.

```python
# middleware.py


class Middleware:
    def __init__(self, app):
        self.app = app

    def add(self, middleware_cls):
        self.app = middleware_cls(self.app)

    def process_request(self, req):
        pass

    def process_response(self, req, resp):
        pass
```

Now, the most important part, the method that handles incoming requests:

```python
# middleware.py


class Middleware:
    ...

    def handle_request(self, request):
        self.process_request(request)
        response = self.app.handle_request(request)
        self.process_response(request, response)

        return response
```

`handle_request` first calls `self.process_request` to do something with the request. Then it delegates the response creation to the app that it is wrapping. Finally, it calls the `process_response` to do something with the response object. Then it simply returns the response upward.

Since middlewares are the first entrypoint to the app, they are now called by a web server (e.g., Gunicorn). Thus, middlewares should implement the WSGI entrypoint interface:

```python
# middleware.py

from webob import Request


class Middleware:
    ...

    def __call__(self, environ, start_response):
        request = Request(environ)
        response = self.app.handle_request(request)
        return response(environ, start_response)

    ...
```

This should look familiar. It's just a copy of the `wsgi_app` function you created in the main `API` class.

You should now have:

```python
# middleware.py

from webob import Request


class Middleware:
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        request = Request(environ)
        response = self.app.handle_request(request)
        return response(environ, start_response)

    def add(self, middleware_cls):
        self.app = middleware_cls(self.app)

    def process_request(self, req):
        pass

    def process_response(self, req, resp):
        pass

    def handle_request(self, request):
        self.process_request(request)
        response = self.app.handle_request(request)
        self.process_response(request, response)

        return response
```

With the `Middleware` class implemented, add it to your main `API` class:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    def __init__(self, templates_dir="templates", static_dir="static"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

        self.whitenoise = WhiteNoise(self.wsgi_app, root=static_dir)

        self.middleware = Middleware(self)

    ...
```

It wraps around `self`, which is a WSGI app.

Now, let's give it the ability to add new middlewares:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    ...

    def add_middleware(self, middleware_cls):
        self.middleware.add(middleware_cls)
```

The only thing left to do is to call this middleware in the entrypoint instead of our own WSGI app:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    ...

    def __call__(self, environ, start_response):
        return self.middleware(environ, start_response)

    ...
```

Again, why are we doing this?

Since we implemented the WSGI entrypoint interface inside the `Middleware` class, we delegated the job of being an entrypoint to the middlewares now.

Run the tests and you will see that all of them pass except for `test_assets_are_served`, which tests the serving of static files. We'll see why this one is failing in a bit.

First, let's manually test this new feature.

## Sanity Check

Go ahead and create a middleware that simply prints a simple message to the console inside *app.py*:

```python
# app.py

from api import API
from middleware import Middleware


app = API()

...

# custom middleware

class SimpleCustomMiddleware(Middleware):
    def process_request(self, req):
        print("Processing request", req.url)

    def process_response(self, req, res):
        print("Processing response", req.url)

app.add_middleware(SimpleCustomMiddleware)
```

Restart Gunicorn and navigate to http://localhost:8000/home. Everything should work just like before. The only exception is that those texts should appear in the console.

Open your console and you should see the following:

```
Processing request http://localhost:8000/home
Processing response http://localhost:8000/home
```

## Static Files

Did you figure out yet why static files don't work?

We stopped using WhiteNoise. It was removed. Instead of calling WhiteNoise, we are now calling the middleware.

To fix this, we need to treat requests for static files differently from all other requests. When a request is coming in for a static file, you should call WhiteNoise. For others, you should call the middleware. The question is how do you distinguish between them?

Right now, request URLs look the same to the `API` class, regardless of whether it's for a static file or something else:

- `http://localhost:8000/main.css`
- `http://localhost:8000/home`

Let's add a root to the URLs of static files -- i.e., `http://localhost:8000/static/main.css`. We can then check if the request path starts with `/static`. If so, we can call WhiteNoise, otherwise we can call the middleware. We should also make sure to remove `/static` from the path; otherwise, WhiteNoise won't find the files.

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    ...

    def __call__(self, environ, start_response):
        path_info = environ["PATH_INFO"]

        if path_info.startswith("/static"):
            environ["PATH_INFO"] = path_info[len("/static"):]
            return self.whitenoise(environ, start_response)

        return self.middleware(environ, start_response)

    ...
```

Now, in the templates, we can call static files like so:

```html
<link href="/static/main.css" type="text/css" rel="stylesheet">
```

Go ahead and change your *index.html* file.

Restart Gunicorn and check that everything, including the static files, is working properly and that the logs are also still appearing in the console.

The last thing to fix is the `test_assets_are_served` test. Since we are using the `/static` base for static files now, change the paths for static files accordingly in *test_bumbo.py*:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware

...

def test_404_is_returned_for_nonexistent_static_file(client):
    assert client.get(f'http://testserver/static/main.css)').status_code == 404


def test_assets_are_served(tmpdir_factory):
    static_dir = tmpdir_factory.mktemp('static')
    _create_static(static_dir)
    api = API(static_dir=str(static_dir))
    client = api.test_session()

    response = client.get(f'http://testserver/static/{FILE_DIR}/{FILE_NAME}')

    assert response.status_code == 200
    assert response.text == FILE_CONTENTS

...
```

Run the tests. They all should pass.

```
======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 14 items

test_bumbo.py ..............                                                                  [100%]

======================================== 14 passed in 0.18s =========================================
```

Great!

You'll use this middleware feature in future chapters to add authentication to apps.

## Code

*api.py*:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    def __init__(self, templates_dir="templates", static_dir="static"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

        self.whitenoise = WhiteNoise(self.wsgi_app, root=static_dir)

        self.middleware = Middleware(self)

    def __call__(self, environ, start_response):
        path_info = environ["PATH_INFO"]

        if path_info.startswith("/static"):
            environ["PATH_INFO"] = path_info[len("/static"):]
            return self.whitenoise(environ, start_response)

        return self.middleware(environ, start_response)

    def wsgi_app(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def add_route(self, path, handler):
        assert path not in self.routes, "Such route already exists."

        self.routes[path] = handler

    def route(self, path):
        def wrapper(handler):
            self.add_route(path, handler)
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler, kwargs = self.find_handler(request_path=request.path)

        try:
            if handler is not None:
                if inspect.isclass(handler):
                    handler = getattr(handler(), request.method.lower(), None)
                    if handler is None:
                        raise AttributeError("Method not allowed", request.method)

                handler(request, response, **kwargs)
            else:
                self.default_response(response)
        except Exception as e:
            if self.exception_handler is None:
                raise e
            else:
                self.exception_handler(request, response, e)

        return response

    def test_session(self, base_url="http://testserver"):
        session = RequestsSession()
        session.mount(prefix=base_url, adapter=RequestsWSGIAdapter(self))
        return session


    def template(self, template_name, context=None):
        if context is None:
            context = {}

        return self.templates_env.get_template(template_name).render(**context)

    def add_exception_handler(self, exception_handler):
        self.exception_handler = exception_handler

    def add_middleware(self, middleware_cls):
        self.middleware.add(middleware_cls)
```

*app.py*:

```python
# app.py

from api import API
from middleware import Middleware


app = API()


# function-based handlers

@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"

@app.route("/about")
def about(request, response):
    response.text = "Hello from the ABOUT page"

@app.route("/hello/{name}")
def greeting(request, response, name):
    response.text = f"Hello, {name}"

@app.route("/sum/{num_1:d}/{num_2:d}")
def sum(request, response, num_1, num_2):
    total = int(num_1) + int(num_2)
    response.text = f"{num_1} + {num_2} = {total}"

@app.route("/template")
def template_handler(req, resp):
    resp.body = app.template(
        "index.html",
        context={"name": "Bumbo", "title": "Best Framework"}
    ).encode()

@app.route("/exception")
def exception_throwing_handler(request, response):
    raise AssertionError("This handler should not be used.")


# class-based handlers

@app.route("/book")
class BooksHandler:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"


# django-like handlers

def handler(req, resp):
    resp.text = "sample"

app.add_route("/sample", handler)


# exception handler

def custom_exception_handler(request, response, exception_cls):
    response.text = str(exception_cls)

app.add_exception_handler(custom_exception_handler)


# custom middleware

class SimpleCustomMiddleware(Middleware):
    def process_request(self, req):
        print("Processing request", req.url)

    def process_response(self, req, res):
        print("Processing response", req.url)

app.add_middleware(SimpleCustomMiddleware)
```

*test_bumbo.py*:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware


FILE_DIR = "css"
FILE_NAME = "main.css"
FILE_CONTENTS = "body {background-color: red}"

# helpers

def _create_static(static_dir):
    asset = static_dir.mkdir(FILE_DIR).join(FILE_NAME)
    asset.write(FILE_CONTENTS)

    return asset


# tests

def test_basic_route_adding(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"


def test_route_overlap_throws_exception(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"

    with pytest.raises(AssertionError):
        @api.route("/home")
        def home2(req, resp):
            resp.text = "YOLO"


def test_bumbo_test_client_can_send_requests(api, client):
    RESPONSE_TEXT = "THIS IS COOL"

    @api.route("/hey")
    def cool(req, resp):
        resp.text = RESPONSE_TEXT

    assert client.get("http://testserver/hey").text == RESPONSE_TEXT


def test_parameterized_route(api, client):
    @api.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
    assert client.get("http://testserver/ashley").text == "hey ashley"


def test_default_404_response(client):
    response = client.get("http://testserver/doesnotexist")

    assert response.status_code == 404
    assert response.text == "Not found."


def test_class_based_handler_get(api, client):
    response_text = "this is a get request"

    @api.route("/book")
    class BookResource:
        def get(self, req, resp):
            resp.text = response_text

    assert client.get("http://testserver/book").text == response_text


def test_class_based_handler_post(api, client):
    response_text = "this is a post request"

    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = response_text

    assert client.post("http://testserver/book").text == response_text


def test_class_based_handler_not_allowed_method(api, client):
    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = "yolo"

    with pytest.raises(AttributeError):
        client.get("http://testserver/book")


def test_alternative_route(api, client):
    response_text = "Alternative way to add a route"

    def home(req, resp):
        resp.text = response_text

    api.add_route("/alternative", home)

    assert client.get("http://testserver/alternative").text == response_text


def test_template(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.body = api.template("index.html", context={"title": "Some Title", "name": "Some Name"}).encode()

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Some Title" in response.text
    assert "Some Name" in response.text


def test_custom_exception_handler(api, client):
    def on_exception(req, resp, exc):
        resp.text = "AttributeErrorHappened"

    api.add_exception_handler(on_exception)

    @api.route("/")
    def index(req, resp):
        raise AttributeError()

    response = client.get("http://testserver/")

    assert response.text == "AttributeErrorHappened"


def test_404_is_returned_for_nonexistent_static_file(client):
    assert client.get(f'http://testserver/static/main.css)').status_code == 404


def test_assets_are_served(tmpdir_factory):
    static_dir = tmpdir_factory.mktemp('static')
    _create_static(static_dir)
    api = API(static_dir=str(static_dir))
    client = api.test_session()

    response = client.get(f'http://testserver/static/{FILE_DIR}/{FILE_NAME}')

    assert response.status_code == 200
    assert response.text == FILE_CONTENTS


def test_middleware_methods_are_called(api, client):
    process_request_called = False
    process_response_called = False

    class CallMiddlewareMethods(Middleware):
        def __init__(self, app):
            super().__init__(app)

        def process_request(self, req):
            nonlocal process_request_called
            process_request_called = True

        def process_response(self, req, resp):
            nonlocal process_response_called
            process_response_called = True

    api.add_middleware(CallMiddlewareMethods)

    @api.route('/')
    def index(req, res):
        res.text = "YOLO"

    client.get('http://testserver/')

    assert process_request_called is True
    assert process_response_called is True
```

# Allowed Methods

##### Chapter 8

In this chapter, we'll add the ability to control which request methods are allowed for the function-based handlers.

## Design

Currently, when you use a class-based handler, you can limit the methods allowed by simply not implementing the method. That's it. Take the following handler as an example:

```python
@app.route("/book")
class BooksResource:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"
```

This handler implements two methods -- `get` and `post`. If you send a `delete` request to this endpoint, it will fail by raising an exception.

```
$ curl -X DELETE http://localhost:8000/book

('Method not allowed', 'DELETE')
```

Here's the code handles that functionality from the `handle_request` method in *api.py*:

```python
if inspect.isclass(handler):
    handler = getattr(handler(), request.method.lower(), None)
    if handler is None:
        raise AttributeError("Method not allowed", request.method)
```

Here, if the request method is not an attribute of the instance of the class-based handler, an `AttributeError` is raised.

How about function-based handlers? How can we limit the allowed methods in them?

At this point, we'd have to write custom code in every handler like so:

```python
@app.route("/home")
def home(request, response):
    if request.method == "get":
        response.text = "Hello from the HOME page"
    else:
        raise AttributeError("Method not allowed.")
```

This is an additional burden for the users of your framework. Let's take it off of their shoulders, shall we?

What do you think about the following solution?

```python
@app.route("/home", allowed_methods=["get"])
def home(request, response):
    response.text = "Hello from the HOME page"
```

The user will simply specify the allowed methods for this endpoint when specifying the route -- and that's it. The framework will handle the job of raising an exception if any other type of request is made.

## Test

With the API designed, write a unit test for it:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware

...

def test_allowed_methods_for_function_based_handlers(api, client):
    @api.route("/home", allowed_methods=["post"])
    def home(req, resp):
        resp.text = "Hello"

    with pytest.raises(AttributeError):
        client.get("http://testserver/home")

    assert client.post("http://testserver/home").text == "Hello"
```

In this test, we created a new handler that only handles POST requests. Then, we asserted that a GET request raises an exception while a POST does not.

Ensure the test fails:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 15 items

test_bumbo.py ..............F                                                                 [100%]

============================================= FAILURES ==============================================
_________________________ test_allowed_methods_for_function_based_handlers __________________________

api = <api.API object at 0x103036290>, client = <requests.sessions.Session object at 0x103036590>

    def test_allowed_methods_for_function_based_handlers(api, client):
>       @api.route("/home", allowed_methods=["post"])
        def home(req, resp):
E       TypeError: route() got an unexpected keyword argument 'allowed_methods'

test_bumbo.py:182: TypeError
=================================== 1 failed, 14 passed in 0.33s ====================================
```

## Implementation

The biggest change that you need to make is associating a particular handler with not only a path but also with a list of allowed methods. As it stands, the `self.routes` dictionary looks like this:

```python
{
    "/home": <function home_handler at 0x1100a70c8>,
    "/about": <function about_handler at 0x1101a80c3>
}
```

In order to store a list of allowed methods, you now need to make the value of this dictionary another dictionary which stores both the handler and a list of allowed methods.

It will then look like this:

```python
{
    "/home": {"handler": <function home_handler at 0x1100a70c8>, "allowed_methods": ["get"]},
    "/about": {"handler": <function about_handler at 0x1101a80c3>, "allowed_methods": ["get", "post"]}
}
```

Then, you will need to make appropriate changes to your `handle_request` method to accommodate these changes. Sounds good? Great. Let's implement it then.

First, you need to change the `route()` method to take the parameter `allowed_methods` and pass it to the `add_route()` method:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    ...

    def route(self, path, allowed_methods=None):
        def wrapper(handler):
            self.add_route(path, handler, allowed_methods)
            return handler

        return wrapper

    ...
```

Next, you need to change `add_route` to take that additional parameter:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    ...

    def add_route(self, path, handler, allowed_methods=None):
        assert path not in self.routes, "Such route already exists."

        if allowed_methods is None:
            allowed_methods = ['get', 'post', 'put', 'patch', 'delete', 'options']

        self.routes[path] = {"handler": handler, "allowed_methods": allowed_methods}

    ...
```

Note the check for `None` here. If `allowed_methods` is not specified by the user, we're allowing all methods to be used. Try writing a test for this on your own. Also note that we're storing `self.routes[path]` now according to the design discussed above.

Next, replace `handler` with `handler_data` in the `find_handler` method:

```python
def find_handler(self, request_path):
    for path, handler_data in self.routes.items():
        parse_result = parse(path, request_path)
        if parse_result is not None:
            return handler_data, parse_result.named

    return None, None
```

Now for the most important part: The `handle_request` method.

First, rename the first `handler` to `handler_data`:

```python
def handle_request(self, request):
    response = Response()

    handler_data, kwargs = self.find_handler(request_path=request.path)

    try:
        if handler is not None:
            if inspect.isclass(handler):
                handler = getattr(handler(), request.method.lower(), None)
                if handler is None:
                    raise AttributeError("Method not allowed", request.method)

            handler(request, response, **kwargs)
        else:
            self.default_response(response)
    except Exception as e:
        if self.exception_handler is None:
            raise e
        else:
            self.exception_handler(request, response, e)
```

Then, in the `try/except` block, if `handler_data` is not `None`, get the `handler` and the `allowed_methods` out of it:

```python
def handle_request(self, request):
    response = Response()

    handler_data, kwargs = self.find_handler(request_path=request.path)

    try:
        if handler_data is not None:
            handler = handler_data["handler"]
            allowed_methods = handler_data["allowed_methods"]
            if inspect.isclass(handler):
                handler = getattr(handler(), request.method.lower(), None)
                if handler is None:
                    raise AttributeError("Method not allowed", request.method)

            handler(request, response, **kwargs)
        else:
            self.default_response(response)
    except Exception as e:
        if self.exception_handler is None:
            raise e
        else:
            self.exception_handler(request, response, e)
```

At this point, everything should work just like before.

Let's take a look at the part that we are most interested in:

```python
if inspect.isclass(handler):
    handler = getattr(handler(), request.method.lower(), None)
    if handler is None:
        raise AttributeError("Method not allowed", request.method)
```

As mentioned above, this `if` clause is used to determine if the `handler` is a class.

Let's add an `else` clause to it to decide what to do if the handler is a function. If it is a function, we can check if the request method is in the list of allowed methods.

```python
if inspect.isclass(handler):
    handler = getattr(handler(), request.method.lower(), None)
    if handler is None:
        raise AttributeError("Method not allowed", request.method)
else:
    if request.method.lower() not in allowed_methods:
        raise AttributeError("Method not allowed", request.method)

handler(request, response, **kwargs)
```

The `handle_request` method should now look like:

```python
def handle_request(self, request):
    response = Response()

    handler_data, kwargs = self.find_handler(request_path=request.path)

    try:
        if handler_data is not None:
            handler = handler_data["handler"]
            allowed_methods = handler_data["allowed_methods"]
            if inspect.isclass(handler):
                handler = getattr(handler(), request.method.lower(), None)
                if handler is None:
                    raise AttributeError("Method not allowed", request.method)
            else:
                if request.method.lower() not in allowed_methods:
                    raise AttributeError("Method not allowed", request.method)

            handler(request, response, **kwargs)
        else:
            self.default_response(response)
    except Exception as e:
        if self.exception_handler is None:
            raise e
        else:
            self.exception_handler(request, response, e)

    return response
```

And that's it. To make sure that it's working as intended, run the tests:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 15 items

test_bumbo.py ...............                                                                 [100%]

======================================== 15 passed in 0.16s =========================================
```

All of the them should now pass. Awesome!

You'll make heavy use of this feature when we build an application with this framework in the future chapters.

See you next time!

## Code

*api.py*:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request, Response
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from middleware import Middleware


class API:
    def __init__(self, templates_dir="templates", static_dir="static"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

        self.whitenoise = WhiteNoise(self.wsgi_app, root=static_dir)

        self.middleware = Middleware(self)

    def __call__(self, environ, start_response):
        path_info = environ["PATH_INFO"]

        if path_info.startswith("/static"):
            environ["PATH_INFO"] = path_info[len("/static"):]
            return self.whitenoise(environ, start_response)

        return self.middleware(environ, start_response)

    def wsgi_app(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def add_route(self, path, handler, allowed_methods=None):
        assert path not in self.routes, "Such route already exists."

        if allowed_methods is None:
            allowed_methods = ['get', 'post', 'put', 'patch', 'delete', 'options']

        self.routes[path] = {"handler": handler, "allowed_methods": allowed_methods}

    def route(self, path, allowed_methods=None):
        def wrapper(handler):
            self.add_route(path, handler, allowed_methods)
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler_data in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler_data, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler_data, kwargs = self.find_handler(request_path=request.path)

        try:
            if handler_data is not None:
                handler = handler_data["handler"]
                allowed_methods = handler_data["allowed_methods"]
                if inspect.isclass(handler):
                    handler = getattr(handler(), request.method.lower(), None)
                    if handler is None:
                        raise AttributeError("Method not allowed", request.method)
                else:
                    if request.method.lower() not in allowed_methods:
                        raise AttributeError("Method not allowed", request.method)

                handler(request, response, **kwargs)
            else:
                self.default_response(response)
        except Exception as e:
            if self.exception_handler is None:
                raise e
            else:
                self.exception_handler(request, response, e)

        return response

    def test_session(self, base_url="http://testserver"):
        session = RequestsSession()
        session.mount(prefix=base_url, adapter=RequestsWSGIAdapter(self))
        return session


    def template(self, template_name, context=None):
        if context is None:
            context = {}

        return self.templates_env.get_template(template_name).render(**context)

    def add_exception_handler(self, exception_handler):
        self.exception_handler = exception_handler

    def add_middleware(self, middleware_cls):
        self.middleware.add(middleware_cls)
```

*test_bumbo.py*:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware


FILE_DIR = "css"
FILE_NAME = "main.css"
FILE_CONTENTS = "body {background-color: red}"

# helpers

def _create_static(static_dir):
    asset = static_dir.mkdir(FILE_DIR).join(FILE_NAME)
    asset.write(FILE_CONTENTS)

    return asset


# tests

def test_basic_route_adding(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"


def test_route_overlap_throws_exception(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"

    with pytest.raises(AssertionError):
        @api.route("/home")
        def home2(req, resp):
            resp.text = "YOLO"


def test_bumbo_test_client_can_send_requests(api, client):
    RESPONSE_TEXT = "THIS IS COOL"

    @api.route("/hey")
    def cool(req, resp):
        resp.text = RESPONSE_TEXT

    assert client.get("http://testserver/hey").text == RESPONSE_TEXT


def test_parameterized_route(api, client):
    @api.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
    assert client.get("http://testserver/ashley").text == "hey ashley"


def test_default_404_response(client):
    response = client.get("http://testserver/doesnotexist")

    assert response.status_code == 404
    assert response.text == "Not found."


def test_class_based_handler_get(api, client):
    response_text = "this is a get request"

    @api.route("/book")
    class BookResource:
        def get(self, req, resp):
            resp.text = response_text

    assert client.get("http://testserver/book").text == response_text


def test_class_based_handler_post(api, client):
    response_text = "this is a post request"

    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = response_text

    assert client.post("http://testserver/book").text == response_text


def test_class_based_handler_not_allowed_method(api, client):
    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = "yolo"

    with pytest.raises(AttributeError):
        client.get("http://testserver/book")


def test_alternative_route(api, client):
    response_text = "Alternative way to add a route"

    def home(req, resp):
        resp.text = response_text

    api.add_route("/alternative", home)

    assert client.get("http://testserver/alternative").text == response_text


def test_template(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.body = api.template("index.html", context={"title": "Some Title", "name": "Some Name"}).encode()

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Some Title" in response.text
    assert "Some Name" in response.text


def test_custom_exception_handler(api, client):
    def on_exception(req, resp, exc):
        resp.text = "AttributeErrorHappened"

    api.add_exception_handler(on_exception)

    @api.route("/")
    def index(req, resp):
        raise AttributeError()

    response = client.get("http://testserver/")

    assert response.text == "AttributeErrorHappened"


def test_404_is_returned_for_nonexistent_static_file(client):
    assert client.get(f'http://testserver/static/main.css)').status_code == 404


def test_assets_are_served(tmpdir_factory):
    static_dir = tmpdir_factory.mktemp('static')
    _create_static(static_dir)
    api = API(static_dir=str(static_dir))
    client = api.test_session()

    response = client.get(f'http://testserver/static/{FILE_DIR}/{FILE_NAME}')

    assert response.status_code == 200
    assert response.text == FILE_CONTENTS


def test_middleware_methods_are_called(api, client):
    process_request_called = False
    process_response_called = False

    class CallMiddlewareMethods(Middleware):
        def __init__(self, app):
            super().__init__(app)

        def process_request(self, req):
            nonlocal process_request_called
            process_request_called = True

        def process_response(self, req, resp):
            nonlocal process_response_called
            process_response_called = True

    api.add_middleware(CallMiddlewareMethods)

    @api.route('/')
    def index(req, res):
        res.text = "YOLO"

    client.get('http://testserver/')

    assert process_request_called is True
    assert process_response_called is True


def test_allowed_methods_for_function_based_handlers(api, client):
    @api.route("/home", allowed_methods=["post"])
    def home(req, resp):
        resp.text = "Hello"

    with pytest.raises(AttributeError):
        client.get("http://testserver/home")

    assert client.post("http://testserver/home").text == "Hello"
```

# Custom Response Class

##### Chapter 9

## Recap

To recap, in the previous chapters, you added the following features to your Python-based web framework:

1. WSGI compatibility
2. Request Handlers
3. Routing -- both simple (`/books/`) and parameterized (`/books/{id}/`)
4. Check for duplicate routes
5. Class-based handlers
6. Unit Tests
7. Test Client
8. Alternative way to add routes (like Django)
9. Support for templates
10. Custom exception handlers
11. Support for static files
12. Middleware
13. Allowed methods for the function-based handlers

Take a few minutes to step back and reflect on what you've learned. Ask yourself the following questions:

1. What were my objectives?
2. How far did I get? How much is left?
3. How was the process? Am I on the right track?

Objectives:

1. Explain what WSGI is and why it's needed
2. Build a basic web framework and run it with Gunicorn, a WSGI-compatible server
3. Develop the core request handlers, routes, and templates
4. Implement class-based route handlers
5. Test your framework with unit tests and practice test-driven development
6. Build a test client to test the API without having to spin up the server
7. Implement custom exception handlers to ensure 404 (Not Found) and 500 (Internal Server Error) errors are handled gracefully
8. Develop solutions for managing static files and middleware in the framework
9. Control allowed methods for your request handlers

------

In this chapter, you'll create a custom response class to make your API more beautiful.

## Design

Do you remember the following handler that we implemented when we added support for templates?

```python
@app.route("/template")
def template_handler(req, resp):
    resp.body = app.template(
        "index.html",
        context={"name": "Bumbo", "title": "Best Framework"}
    ).encode()
```

At the end of the chapter, I mentioned that it was not the most elegant solution, which is even more obvious when you try to return JSON as a response:

```python
import json


@app.route("/json")
def json_handler(req, resp):
    response_data = {"name": "data", "type": "JSON"}

    resp.body = json.dumps(response_data).encode()
    resp.content_type = "application/json"
```

The encoding, JSON-dumps, and content-types are fairly ugly. So, let's create a custom class wrapper around the `WebOb.Response` class and add custom properties to it.

With their help, returning an HTML response will look like this:

```python
@app.route("/template")
def template_handler(req, resp):
    resp.html = app.template("index.html", context={"name": "Bumbo", "title": "Best Framework"})
```

A JSON response will look like this:

```python
@app.route("/json")
def json_handler(req, resp):
    resp.json = {"name": "data", "type": "JSON"}
```

And a plain text response will look like this:

```python
@app.route("/text")
def plain_text_handler(req, resp):
    resp.text = "This is a simple text"
```

Much simpler, right? No need to encode and set content types manually. Look good?

## Test

Add some tests:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware

...

def test_json_response_helper(api, client):
    @api.route("/json")
    def json_handler(req, resp):
        resp.json = {"name": "bubmo"}

    response = client.get("http://testserver/json")
    json_body = response.json()

    assert response.headers["Content-Type"] == "application/json"
    assert json_body["name"] == "bubmo"


def test_html_response_helper(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.html = api.template("index.html", context={"title": "Best Title", "name": "Best Name"})

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Best Title" in response.text
    assert "Best Name" in response.text


def test_text_response_helper(api, client):
    response_text = "Just Plain Text"

    @api.route("/text")
    def text_handler(req, resp):
        resp.text = response_text

    response = client.get("http://testserver/text")

    assert "text/plain" in response.headers["Content-Type"]
    assert response.text == response_text


def test_manually_setting_body(api, client):
    @api.route("/body")
    def text_handler(req, resp):
        resp.body = b"Byte Body"
        resp.content_type = "text/plain"

    response = client.get("http://testserver/body")

    assert "text/plain" in response.headers["Content-Type"]
    assert response.text == "Byte Body"
```

Each of these test that the new properties of the response class are working correctly.

The first three should fail:

```
$(venv) pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 19 items

test_bumbo.py ...............FFF.                                                             [100%]

============================================= FAILURES ==============================================
_____________________________________ test_json_response_helper _____________________________________

api = <api.API object at 0x10825c610>, client = <requests.sessions.Session object at 0x10826a110>

    def test_json_response_helper(api, client):
        @api.route("/json")
        def json_handler(req, resp):
            resp.json = {"name": "bubmo"}

        response = client.get("http://testserver/json")
        json_body = response.json()

>       assert response.headers["Content-Type"] == "application/json"
E       AssertionError: assert 'text/html; charset=UTF-8' == 'application/json'
E         - text/html; charset=UTF-8
E         + application/json

test_bumbo.py:200: AssertionError
_____________________________________ test_html_response_helper _____________________________________

api = <api.API object at 0x1081766d0>, client = <requests.sessions.Session object at 0x108176590>

    def test_html_response_helper(api, client):
        @api.route("/html")
        def html_handler(req, resp):
            resp.html = api.template("index.html", context={"title": "Best Title", "name": "Best Name"})

        response = client.get("http://testserver/html")

        assert "text/html" in response.headers["Content-Type"]
>       assert "Best Title" in response.text
E       AssertionError: assert 'Best Title' in ''
E        +  where '' = <Response [200]>.text

test_bumbo.py:212: AssertionError
_____________________________________ test_text_response_helper _____________________________________

api = <api.API object at 0x1082494d0>, client = <requests.sessions.Session object at 0x108249850>

    def test_text_response_helper(api, client):
        response_text = "Just Plain Text"

        @api.route("/text")
        def text_handler(req, resp):
            resp.text = response_text

        response = client.get("http://testserver/text")

>       assert "text/plain" in response.headers["Content-Type"]
E       AssertionError: assert 'text/plain' in 'text/html; charset=UTF-8'

test_bumbo.py:225: AssertionError
=================================== 3 failed, 16 passed in 0.34s ====================================
```

## Implementation

First, create a `response.py` file inside the "bumbo" folder:

```
$ touch response.py
```

Inside it, create a `Response` class with the necessary properties:

```python
# response.py


class Response:
    def __init__(self):
        self.json = None
        self.html = None
        self.text = None
        self.content_type = None
        self.body = b''
        self.status_code = 200
```

Recall that this response class is a wrapper around the `WebOb.Response` class. Thus, when the instances of this class are called, you need to return an instance of a `WebOb` class with the given properties:

```python
# response.py

from webob import Response as WebObResponse


class Response:
    def __init__(self):
        ...

    def __call__(self, environ, start_response):
        response = WebObResponse(
            body=self.body, content_type=self.content_type, status=self.status_code
        )
        return response(environ, start_response)
```

Now the most important part: Before creating a `WebObResponse` instance, you need to set `self.body` and `self.content_type` according to the values in `self.json`, `self.html`, or `self.text`.

For that purpose, let's keep it simple and lump everything into a single method:

```python
# response.py

import json

from webob import Response as WebObResponse


class Response:
    ...

    def set_body_and_content_type(self):
        if self.json is not None:
            self.body = json.dumps(self.json).encode('UTF-8')
            self.content_type = "application/json"

        if self.html is not None:
            self.body = self.html.encode()
            self.content_type = "text/html"

        if self.text is not None:
            self.body = self.text
            self.content_type = "text/plain"
```

This new method should be fairly self-explanatory: The body and the content type are set based on the `self.json`, `self.html`, and `self.text` properties.

Now, you just need to call this method before the `WebObResponse` instance creation:

```python
# response.py

import json

from webob import Response as WebObResponse


class Response:
    ...

    def __call__(self, environ, start_response):
        self.set_body_and_content_type()

        response = WebObResponse(
            body=self.body, content_type=self.content_type, status=self.status_code
        )
        return response(environ, start_response)

    ...
```

Lastly, in *api.py* import this `Response` class instead of the one from `webob`:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from response import Response
from middleware import Middleware

...
```

Make sure that all tests pass:

```
(venv)$ pytest test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 19 items

test_bumbo.py ...................                                                             [100%]

======================================== 19 passed in 0.21s =========================================
```

How's test coverage looking?

```
(venv)$ pytest --cov=. test_bumbo.py

======================================== test session starts ========================================
platform darwin -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0
rootdir: /Users/michael.herman/repos/testdriven/python-web-framework/bumbo
plugins: cov-2.8.1
collected 19 items

test_bumbo.py ...................                                                             [100%]

---------- coverage: platform darwin, python 3.7.4-final-0 -----------
Name            Stmts   Miss  Cover
-----------------------------------
api.py             78      1    99%
middleware.py      19      2    89%
response.py        24      0   100%
-----------------------------------
TOTAL             121      3    98%


======================================== 19 passed in 0.28s =========================================
```

Looks great!

Now, let's go ahead and manually test this new feature by adding the following handlers to *app.py*:

```python
# app.py

from api import API
from middleware import Middleware


app = API()

...

@app.route("/template")
def template_handler(req, resp):
    resp.html = app.template("index.html", context={"name": "Bumbo", "title": "Best Framework"})

@app.route("/json")
def json_handler(req, resp):
    resp.json = {"name": "data", "type": "JSON"}

@app.route("/text")
def text_handler(req, resp):
    resp.text = "This is a simple text"

...
```

Remember to delete the old `/template` handler.

Restart Gunicorn and manually test the new handlers by navigating to http://localhost:8000/template, http://localhost:8000/json, and http://localhost:8000/text. Make sure that each of them looks as it's supposed to.

## Conclusion

At this point, you have built the necessary parts of the framework to start building an app with it. However, it is not installable yet. Thus, in the next chapter, you'll publish the framework to PyPI so that our users can easily install it by simply running `pip install bumbo`.

See you there!

## Code

*api.py*:

```python
# api.py

import os
import inspect

from parse import parse
from webob import Request
from requests import Session as RequestsSession
from wsgiadapter import WSGIAdapter as RequestsWSGIAdapter
from jinja2 import Environment, FileSystemLoader
from whitenoise import WhiteNoise

from response import Response
from middleware import Middleware


class API:
    def __init__(self, templates_dir="templates", static_dir="static"):
        self.routes = {}

        self.templates_env = Environment(
            loader=FileSystemLoader(os.path.abspath(templates_dir))
        )

        self.exception_handler = None

        self.whitenoise = WhiteNoise(self.wsgi_app, root=static_dir)

        self.middleware = Middleware(self)

    def __call__(self, environ, start_response):
        path_info = environ["PATH_INFO"]

        if path_info.startswith("/static"):
            environ["PATH_INFO"] = path_info[len("/static"):]
            return self.whitenoise(environ, start_response)

        return self.middleware(environ, start_response)

    def wsgi_app(self, environ, start_response):
        request = Request(environ)

        response = self.handle_request(request)

        return response(environ, start_response)

    def add_route(self, path, handler, allowed_methods=None):
        assert path not in self.routes, "Such route already exists."

        if allowed_methods is None:
            allowed_methods = ['get', 'post', 'put', 'patch', 'delete', 'options']

        self.routes[path] = {"handler": handler, "allowed_methods": allowed_methods}

    def route(self, path, allowed_methods=None):
        def wrapper(handler):
            self.add_route(path, handler, allowed_methods)
            return handler

        return wrapper

    def default_response(self, response):
        response.status_code = 404
        response.text = "Not found."

    def find_handler(self, request_path):
        for path, handler_data in self.routes.items():
            parse_result = parse(path, request_path)
            if parse_result is not None:
                return handler_data, parse_result.named

        return None, None

    def handle_request(self, request):
        response = Response()

        handler_data, kwargs = self.find_handler(request_path=request.path)

        try:
            if handler_data is not None:
                handler = handler_data["handler"]
                allowed_methods = handler_data["allowed_methods"]
                if inspect.isclass(handler):
                    handler = getattr(handler(), request.method.lower(), None)
                    if handler is None:
                        raise AttributeError("Method not allowed", request.method)
                else:
                    if request.method.lower() not in allowed_methods:
                        raise AttributeError("Method not allowed", request.method)

                handler(request, response, **kwargs)
            else:
                self.default_response(response)
        except Exception as e:
            if self.exception_handler is None:
                raise e
            else:
                self.exception_handler(request, response, e)

        return response

    def test_session(self, base_url="http://testserver"):
        session = RequestsSession()
        session.mount(prefix=base_url, adapter=RequestsWSGIAdapter(self))
        return session


    def template(self, template_name, context=None):
        if context is None:
            context = {}

        return self.templates_env.get_template(template_name).render(**context)

    def add_exception_handler(self, exception_handler):
        self.exception_handler = exception_handler

    def add_middleware(self, middleware_cls):
        self.middleware.add(middleware_cls)
```

*app.py*:

```python
# app.py

from api import API
from middleware import Middleware


app = API()


# function-based handlers

@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"

@app.route("/about")
def about(request, response):
    response.text = "Hello from the ABOUT page"

@app.route("/hello/{name}")
def greeting(request, response, name):
    response.text = f"Hello, {name}"

@app.route("/sum/{num_1:d}/{num_2:d}")
def sum(request, response, num_1, num_2):
    total = int(num_1) + int(num_2)
    response.text = f"{num_1} + {num_2} = {total}"

@app.route("/exception")
def exception_throwing_handler(request, response):
    raise AssertionError("This handler should not be used.")

@app.route("/template")
def template_handler(req, resp):
    resp.html = app.template("index.html", context={"name": "Bumbo", "title": "Best Framework"})


@app.route("/json")
def json_handler(req, resp):
    resp.json = {"name": "data", "type": "JSON"}


@app.route("/text")
def text_handler(req, resp):
    resp.text = "This is a simple text"


# class-based handlers

@app.route("/book")
class BooksHandler:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"


# django-like handlers

def handler(req, resp):
    resp.text = "sample"

app.add_route("/sample", handler)


# exception handler

def custom_exception_handler(request, response, exception_cls):
    response.text = str(exception_cls)

app.add_exception_handler(custom_exception_handler)


# custom middleware

class SimpleCustomMiddleware(Middleware):
    def process_request(self, req):
        print("Processing request", req.url)

    def process_response(self, req, res):
        print("Processing response", req.url)

app.add_middleware(SimpleCustomMiddleware)
```

*response.py*:

```python
# response.py

import json

from webob import Response as WebObResponse


class Response:
    def __init__(self):
        self.json = None
        self.html = None
        self.text = None
        self.content_type = None
        self.body = b''
        self.status_code = 200

    def __call__(self, environ, start_response):
        self.set_body_and_content_type()

        response = WebObResponse(
            body=self.body, content_type=self.content_type, status=self.status_code
        )
        return response(environ, start_response)

    def set_body_and_content_type(self):
        if self.json is not None:
            self.body = json.dumps(self.json).encode('UTF-8')
            self.content_type = "application/json"

        if self.html is not None:
            self.body = self.html.encode()
            self.content_type = "text/html"

        if self.text is not None:
            self.body = self.text
            self.content_type = "text/plain"
```

*test_bumbo.py*:

```python
# test_bumbo.py

import pytest

from api import API
from middleware import Middleware


FILE_DIR = "css"
FILE_NAME = "main.css"
FILE_CONTENTS = "body {background-color: red}"

# helpers

def _create_static(static_dir):
    asset = static_dir.mkdir(FILE_DIR).join(FILE_NAME)
    asset.write(FILE_CONTENTS)

    return asset


# tests

def test_basic_route_adding(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"


def test_route_overlap_throws_exception(api):
    @api.route("/home")
    def home(req, resp):
        resp.text = "YOLO"

    with pytest.raises(AssertionError):
        @api.route("/home")
        def home2(req, resp):
            resp.text = "YOLO"


def test_bumbo_test_client_can_send_requests(api, client):
    RESPONSE_TEXT = "THIS IS COOL"

    @api.route("/hey")
    def cool(req, resp):
        resp.text = RESPONSE_TEXT

    assert client.get("http://testserver/hey").text == RESPONSE_TEXT


def test_parameterized_route(api, client):
    @api.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
    assert client.get("http://testserver/ashley").text == "hey ashley"


def test_default_404_response(client):
    response = client.get("http://testserver/doesnotexist")

    assert response.status_code == 404
    assert response.text == "Not found."


def test_class_based_handler_get(api, client):
    response_text = "this is a get request"

    @api.route("/book")
    class BookResource:
        def get(self, req, resp):
            resp.text = response_text

    assert client.get("http://testserver/book").text == response_text


def test_class_based_handler_post(api, client):
    response_text = "this is a post request"

    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = response_text

    assert client.post("http://testserver/book").text == response_text


def test_class_based_handler_not_allowed_method(api, client):
    @api.route("/book")
    class BookResource:
        def post(self, req, resp):
            resp.text = "yolo"

    with pytest.raises(AttributeError):
        client.get("http://testserver/book")


def test_alternative_route(api, client):
    response_text = "Alternative way to add a route"

    def home(req, resp):
        resp.text = response_text

    api.add_route("/alternative", home)

    assert client.get("http://testserver/alternative").text == response_text


def test_template(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.body = api.template("index.html", context={"title": "Some Title", "name": "Some Name"}).encode()

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Some Title" in response.text
    assert "Some Name" in response.text


def test_custom_exception_handler(api, client):
    def on_exception(req, resp, exc):
        resp.text = "AttributeErrorHappened"

    api.add_exception_handler(on_exception)

    @api.route("/")
    def index(req, resp):
        raise AttributeError()

    response = client.get("http://testserver/")

    assert response.text == "AttributeErrorHappened"


def test_404_is_returned_for_nonexistent_static_file(client):
    assert client.get(f'http://testserver/static/main.css)').status_code == 404


def test_assets_are_served(tmpdir_factory):
    static_dir = tmpdir_factory.mktemp('static')
    _create_static(static_dir)
    api = API(static_dir=str(static_dir))
    client = api.test_session()

    response = client.get(f'http://testserver/static/{FILE_DIR}/{FILE_NAME}')

    assert response.status_code == 200
    assert response.text == FILE_CONTENTS


def test_middleware_methods_are_called(api, client):
    process_request_called = False
    process_response_called = False

    class CallMiddlewareMethods(Middleware):
        def __init__(self, app):
            super().__init__(app)

        def process_request(self, req):
            nonlocal process_request_called
            process_request_called = True

        def process_response(self, req, resp):
            nonlocal process_response_called
            process_response_called = True

    api.add_middleware(CallMiddlewareMethods)

    @api.route('/')
    def index(req, res):
        res.text = "YOLO"

    client.get('http://testserver/')

    assert process_request_called is True
    assert process_response_called is True


def test_allowed_methods_for_function_based_handlers(api, client):
    @api.route("/home", allowed_methods=["post"])
    def home(req, resp):
        resp.text = "Hello"

    with pytest.raises(AttributeError):
        client.get("http://testserver/home")

    assert client.post("http://testserver/home").text == "Hello"


def test_json_response_helper(api, client):
    @api.route("/json")
    def json_handler(req, resp):
        resp.json = {"name": "bubmo"}

    response = client.get("http://testserver/json")
    json_body = response.json()

    assert response.headers["Content-Type"] == "application/json"
    assert json_body["name"] == "bubmo"


def test_html_response_helper(api, client):
    @api.route("/html")
    def html_handler(req, resp):
        resp.html = api.template("index.html", context={"title": "Best Title", "name": "Best Name"})

    response = client.get("http://testserver/html")

    assert "text/html" in response.headers["Content-Type"]
    assert "Best Title" in response.text
    assert "Best Name" in response.text


def test_text_response_helper(api, client):
    response_text = "Just Plain Text"

    @api.route("/text")
    def text_handler(req, resp):
        resp.text = response_text

    response = client.get("http://testserver/text")

    assert "text/plain" in response.headers["Content-Type"]
    assert response.text == response_text


def test_manually_setting_body(api, client):
    @api.route("/body")
    def text_handler(req, resp):
        resp.body = b"Byte Body"
        resp.content_type = "text/plain"

    response = client.get("http://testserver/body")

    assert "text/plain" in response.headers["Content-Type"]
    assert response.text == "Byte Body"
```

# PyPI

##### Chapter 10

You've come a long way and built the initial version of your framework. However, for you and your users to be able to build something with it, it needs to be installable. To do that, you need to upload the framework to the [Python Packaging Index](https://pypi.org/) (PyPI) where third party Python packages are hosted. Once you do that, every single person in the world will be able to install the framework to their local machine by simply running `pip install bumbo`. Sounds good? Let's get started then.

## Preparation

First, let's review the structure of the framework:

```
├── api.py
├── app.py
├── conftest.py
├── middleware.py
├── response.py
├── static
│   └── main.css
├── templates
│   └── index.html
└── test_bumbo.py
```

Note that "app.py", "templates" and "static" have no use for the users of the framework. Those files and folders were created for testing purposes only. As it stands, the most important files are *api.py*, *middleware.py*, and *response.py*. *test_bumbo.py* and *conftest.py* are used for unit tests. So, let's separate those files from each other.

Go ahead and create a top level folder called "bumbo":

```
(venv)$ mkdir bumbo
```

Move *api.py*, *middleware.py*, and *response.py* into this folder:

```
(venv)$ mv api.py middleware.py response.py bumbo/
```

Now the structure of the project looks like this:

```
├── app.py
├── bumbo
│   ├── api.py
│   ├── middleware.py
│   └── response.py
├── conftest.py
├── static
│   └── main.css
├── templates
│   └── index.html
└── test_bumbo.py
```

Since *api.py*, *middleware.py*, and *response.py* are not in the project root anymore, you need to change two imports inside *api.py*.

Change:

```
from response import Response
from middleware import Middleware
```

To:

```python
from .response import Response
from .middleware import Middleware
```

Now, after installing the framework, your users will be able to import the necessary classes like so:

```
from bumbo.api import API
```

However, there is a catch. Have you found it? That `bumbo` folder is not a Python package yet and thus it cannot be imported from.

To fix that, create a *__init__.py* file inside it:

```
(venv)$ touch bumbo/__init__.py
```

Now that your project structure is ready, you need to let PyPI know some basic information about the project -- like its name, the version, and what other packages it requires. That information is provided in the form of [setup.py](https://stackoverflow.com/questions/1471994/what-is-setup-py). We'll base our *setup.py* file on [setup.py for humans](https://github.com/navdeep-G/setup.py):

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import io
import os

from setuptools import find_packages, setup

# Package meta-data.
NAME = "bumbo"
DESCRIPTION = "Bumbo Python Web Framework built for learning purposes."
EMAIL = "jrahmonov2@gmail.com"
AUTHOR = "Jahongir Rahmonov"
REQUIRES_PYTHON = ">=3.6.0"
VERSION = "0.0.1"

# Which packages are required for this module to be executed?
REQUIRED = [
    "Jinja2==2.10.3",
    "parse==1.12.1",
    "requests==2.22.0",
    "requests-wsgi-adapter==0.4.1",
    "WebOb==1.8.5",
    "whitenoise==4.1.4",
]

# The rest you shouldn't have to touch too much :)

here = os.path.abspath(os.path.dirname(__file__))

# Import the README and use it as the long-description.
# Note: this will only work if 'README.md' is present in your MANIFEST.in file!
try:
    with io.open(os.path.join(here, "README.md"), encoding="utf-8") as f:
        long_description = "\n" + f.read()
except FileNotFoundError:
    long_description = DESCRIPTION

# Load the package's __version__.py module as a dictionary.
about = {}
if not VERSION:
    project_slug = NAME.lower().replace("-", "_").replace(" ", "_")
    with open(os.path.join(here, project_slug, "__version__.py")) as f:
        exec(f.read(), about)
else:
    about["__version__"] = VERSION


# Where the magic happens:
setup(
    name=NAME,
    version=about["__version__"],
    description=DESCRIPTION,
    long_description=long_description,
    long_description_content_type="text/markdown",
    author=AUTHOR,
    author_email=EMAIL,
    python_requires=REQUIRES_PYTHON,
    packages=find_packages(exclude=["test_*"]),
    install_requires=REQUIRED,
    include_package_data=True,
    license="MIT",
    classifiers=[
        "Programming Language :: Python :: 3.6",
    ],
    setup_requires=["wheel"],
)
```

Add this file to the project root.

I know it may seem a little confusing but it's actually fairly simple. The only thing that you will need to change is the `Package meta-data` part:

1. The project name name needs to be changed since I already uploaded a package with the name `bumbo`. Try customizing it by adding your name to it -- `bumbo_alex` or `bumbo_kate`, for example. Name it however you want. Just make it unique.
2. Change the email and the author name as well, of course.

You shouldn't have to touch the rest. But make sure to take a look at the bottom function called `setup()`. This is the function responsible for generating the necessary files for PyPI as you will see later.

## Publishing to PyPI

You are finally ready to upload your framework to PyPI. To do that, you will need an account on PyPI. So, go ahead and register one [here](https://pypi.org/account/register/).

You will also need a package called [twine](https://twine.readthedocs.io/), which is a utility for publishing Python packages on PyPI:

```
(venv)$ pip install twine
```

PyPI doesn't store packages as plain source code. It stores them as archives and [Python wheels](https://wheel.readthedocs.io/en/stable/). So, before you can upload it to PyPI, you need to build those archives and wheels:

```
(venv)$ python setup.py sdist bdist_wheel
```

Run it and you will see a `dist` folder created that includes an archive and a wheel file.

You can run the following command to see if they were created correctly:

```
(venv)$ twine check dist/*
```

If all the checks passed, you can upload the framework to PyPI:

```
(venv)$ twine upload dist/*
```

You will be prompted for your username and password.

Once uploaded, navigate to https://pypi.org/project/PACKAGE_NAME/ and you will see your beloved framework there. How does it feel? Great! Congrats with publishing your package to PyPI.

You can now easily install your framework with `pip`:

```
(venv)$ pip install <PACKAGE_NAME>
```

## Documentation

Documentation is just as important, if not more, as the framework you built. No matter how awesome your framework is, if users don't know how to use it, why build it in the first place? So, let's add some documentation to the project.

Create a *README.md* file:

```
(venv)$ touch README.md
```

It's a [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) file so that Github and PyPI can interpret it correctly.

First, put the title in the README:

```
# Bumbo: Python Web Framework built for learning purposes
```

Next, feel free to add some flair with some badges from [Shields.io](https://shields.io/).

For example:

```markdown
# Bumbo: Python Web Framework built for learning purposes

![purpose](https://img.shields.io/badge/purpose-learning-green.svg)
![PyPI](https://img.shields.io/pypi/v/bumbo.svg)
```

As for the actual content, write a few sentences about what the project is about:

```
# Bumbo: Python Web Framework built for learning purposes

![purpose](https://img.shields.io/badge/purpose-learning-green.svg)
![PyPI](https://img.shields.io/pypi/v/bumbo.svg)

Bumbo is a Python web framework built for learning purposes.

It's a WSGI framework and can be used with any WSGI application server such as Gunicorn.
```

Next, add installation instructions:

```
# Bumbo: Python Web Framework built for learning purposes

![purpose](https://img.shields.io/badge/purpose-learning-green.svg)
![PyPI](https://img.shields.io/pypi/v/bumbo.svg)

Bumbo is a Python web framework built for learning purposes.

It's a WSGI framework and can be used with any WSGI application server such as Gunicorn.

## Installation

​```shell
pip install bumbo
​```
```

Then, add the most important part -- "How to use it":

```
# Bumbo: Python Web Framework built for learning purposes

![purpose](https://img.shields.io/badge/purpose-learning-green.svg)
![PyPI](https://img.shields.io/pypi/v/bumbo.svg)

Bumbo is a Python web framework built for learning purposes.

It's a WSGI framework and can be used with any WSGI application server such as Gunicorn.

## Installation

​```shell
pip install bumbo
​```

## How to use it

### Basic usage:

​```python
from bumbo.api import API

app = API()


@app.route("/home")
def home(request, response):
    response.text = "Hello from the HOME page"


@app.route("/hello/{name}")
def greeting(request, response, name):
    response.text = f"Hello, {name}"


@app.route("/book")
class BooksResource:
    def get(self, req, resp):
        resp.text = "Books Page"

    def post(self, req, resp):
        resp.text = "Endpoint to create a book"


@app.route("/template")
def template_handler(req, resp):
    resp.body = app.template(
        "index.html", context={"name": "Bumbo", "title": "Best Framework"}).encode()
​```

### Unit Tests

The recommended way of writing unit tests is with [pytest](https://docs.pytest.org/en/latest/). There are two built in fixtures
that you may want to use when writing unit tests with Bumbo. The first one is `app` which is an instance of the main `API` class:

​```python
def test_route_overlap_throws_exception(app):
    @app.route("/")
    def home(req, resp):
        resp.text = "Welcome Home."

    with pytest.raises(AssertionError):
        @app.route("/")
        def home2(req, resp):
            resp.text = "Welcome Home2."
​```

The other one is `client` that you can use to send HTTP requests to your handlers. It is based on the famous [requests](http://docs.python-requests.org/en/master/) and it should feel very familiar:

​```python
def test_parameterized_route(app, client):
    @app.route("/{name}")
    def hello(req, resp, name):
        resp.text = f"hey {name}"

    assert client.get("http://testserver/matthew").text == "hey matthew"
​```

## Templates

The default folder for templates is `templates`. You can change it when initializing the main `API()` class:

​```python
app = API(templates_dir="templates_dir_name")
​```

Then you can use HTML files in that folder like so in a handler:

​```python
@app.route("/show/template")
def handler_with_template(req, resp):
    resp.html = app.template(
        "example.html", context={"title": "Awesome Framework", "body": "welcome to the future!"})
​```

## Static Files

Just like templates, the default folder for static files is `static` and you can override it:

​```python
app = API(static_dir="static_dir_name")
​```

Then you can use the files inside this folder in HTML files:

​```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>{{title}}</title>

  <link href="/static/main.css" rel="stylesheet" type="text/css">
</head>

<body>
    <h1>{{body}}</h1>
    <p>This is a paragraph</p>
</body>
</html>
​```

### Middleware

You can create custom middleware classes by inheriting from the `bumbo.middleware.Middleware` class and overriding its two methods
that are called before and after each request:

​```python
from bumbo.api import API
from bumbo.middleware import Middleware


app = API()


class SimpleCustomMiddleware(Middleware):
    def process_request(self, req):
        print("Before dispatch", req.url)

    def process_response(self, req, res):
        print("After dispatch", req.url)


app.add_middleware(SimpleCustomMiddleware)
​```
```

With the most critical parts of the documentation done, you can add `How to contribute`, `Contributors` and similar sections next. But I'll leave that to you.

## Publish a New Version

Now that you have good documentation, go ahead and release the second version of the framework to PyPI.

To do that, you first need to change the version in `setup.py`. Find the `VERSION` in the `Package meta-data` section and change it from `0.0.1` to `0.0.2`.

Delete the old `0.0.1` version files and folders in the "dist" directory.

Then, build the new version and upload it to PyPI:

```
(venv)$ python setup.py sdist bdist_wheel
(venv)$ twine upload dist/*
```

And there you go. Your users can now install the new version of the framework. Make sure to check out the second version in PyPI by navigating to https://pypi.org/project/PACKAGE_NAME/. You should see the documentation you just wrote.

## Conclusion

In this chapter, you learned how to prepare a Python package for publishing to PyPI. You then uploaded the framework to PyPI and released a new version. You are now ready to build something with the help of your own web framework in the next chapter.

See you there!

# Web App

##### Chapter 11

Up to this point, you've built most of the critical features of the Python web framework and uploaded it to PyPI so that it's installable via `pip`. In this chapter, you'll build a small web app with the help of your own framework to see what it feels like to use it.

The web app is a basic CRUD app for books. You can see a list of books, you can add a new book and you can delete a book. Adding and deleting will require authorization so we'll add token authentication and authorization functionality as well. After you are done with this web app, you will have used all the features of the framework.

Sound good? Let's get started then.

## Setup

First, create a folder for the project, and then create and activate a virtual environment:

```
(venv)$ mkdir webapp && cd webapp
$ python3.7 -m venv venv
$ source venv/bin/activate
```

> This is a separate project. Do *not* create the files inside the framework project.

Install your framework:

```
(venv)$ pip install bumbo
```

> You should have named your framework differently, so replace `bumbo` with that name.

Create an *app.py* file, and then import the main `API` class from the framework and initialize it:

```
# app.py

from bumbo.api import API


app = API()
```

Now, create a handler for the index page:

```
# app.py

from bumbo.api import API


app = API()


@app.route("/", allowed_methods=["get"])
def index(req, resp):
    resp.html = app.template("index.html")
```

As you can see, it renders a template file called *index.html*. So, create a "templates" folder along with an *index.html* file:

```
(venv)$ mkdir templates
(venv)$ touch templates/index.html
```

For now, you can write whatever you like in *index.html* -- like `<h1>hello, world!</h1>`, for example -- because you are just testing it.

Before running it, install Gunicorn:

```
(venv)$ pip install gunicorn
```

Now, run the project you just created:

```
(venv)$ gunicorn app:app
```

If you see an error, try to deactivate the virtualenv and then reactivate.

In your browser of choice, navigate to http://localhost:8000/ and you should see the rendered template that you just created.

## List of Books

Now, in the index page, you need to show the list of books. However, you don't have books in the database yet. You don't even have the database itself, for that matter. For this example, to keep things simple, you will store the books in memory, but you can use an ORM -- like [SQLAlchemy](https://www.sqlalchemy.org/) or [Peewee](http://docs.peewee-orm.com/) -- along with a database instead if you'd like.

So, go ahead and create a *models.py* file in the project root:

```
(venv)$ touch models.py
```

And put the following inside:

```python
# models.py

from typing import NamedTuple


class Book(NamedTuple):
    id: int
    name: str
    author: str
```

Using a `NamedTuple` is a quick and easy way of creating simple classes in Python without having to use the `__init__` method and properties. Here, you just created a named tuple with three properties -- `id`, `name`, and `author`.

Now that we have a model class, we need a storage class that interacts with the model class, which deals with the creating, storing, listing, and deleting of books.

Create a *storage.py* file:

```python
# storage.py

from models import Book


class BookStorage:
    _id = 0

    def __init__(self):
        self._books = []

    def all(self):
        return [book._asdict() for book in self._books]

    def get(self, id: int):
        for book in self._books:
            if book.id == id:
                return book

        return None

    def create(self, **kwargs):
        self._id += 1
        kwargs["id"] = self._id
        book = Book(**kwargs)
        self._books.append(book)
        return book

    def delete(self, id):
        for ind, book in enumerate(self._books):
            if book.id == id:
                del self._books[ind]
```

Again, this storage file will serve as a database that will store all the data about books in memory. It has methods to list, get, create, and delete books. Initialize this storage class in *app.py* right after the `API` class initialization and create one book in order to have an example to work with:

```python
# app.py

from bumbo.api import API

from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")


@app.route("/", allowed_methods=["get"])
def index(req, resp):
    resp.html = app.template("index.html")
```

Now in the index handler, you can get the list of books and send them to the template as a context:

```python
# app.py

from bumbo.api import API

from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")


@app.route("/", allowed_methods=["get"])
def index(req, resp):
    books = book_storage.all()
    resp.html = app.template("index.html", context={"books": books})
```

You can now edit the template to show the list of books in a table:

```python
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>List of Books</title>
  <link href="/static/main.css" type="text/css" rel="stylesheet">
</head>
<body>

<table>
  <tr>
    <th>ID</th>
    <th>Name</th>
    <th>Author</th>
    <th>Actions</th>
  </tr>

  {% for book in books %}
    <tr>
      <td>{{ book.id }}</td>
      <td>{{ book.name }}</td>
      <td>{{ book.author }}</td>
      <td><button data-id="{{book.id}}" class="delete-button">Delete</button></td>
    </tr>
  {% endfor %}
</table>

</body>
</html>
```

Note the *main.css* that was linked in the `<head>`. That's where we'll put all the CSS styles. So, create the "static" folder first and then create *main.css*:

```
(venv)$ mkdir static
(venv)$ touch static/main.css
```

Put the following inside *main.css* to make the table look better:

```
table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
```

Restart Gunicorn and navigate to http://localhost:8000/. You should see:

![app](https://testdriven.io/static/images/courses/python-web-framework/index.png)

This is the only publicly available page. Deleting and creating books will require authorization. So, for authorization, you'll:

1. Use a single static token for everyone to keep things simple (but feel free to implement a full-blown auth system)
2. Create a middleware that grabs the token from the header and attaches it to the request
3. Create a decorator, for the create and delete handlers, that checks if a user is logged in or not
4. Also create a login handler that simply returns the static token

## Auth

Let's deal with all auth-related things in a single place. So, go ahead and create an *auth.py* file for that purpose:

```
(venv)$ touch auth.py
```

In *auth.py*, create a constant variable called `STATIC_TOKEN`:

```
# auth.py


STATIC_TOKEN = "ae4CMvqBe2"
```

Then, in *app.py*, create a login handler that simply returns this token in a JSON object:

```python
# app.py

from bumbo.api import API

from auth import STATIC_TOKEN
from storage import BookStorage

...

@app.route("/login", allowed_methods=["post"])
def login(req, resp):
    resp.json = {"token": STATIC_TOKEN}
```

Now, write the token middleware in `auth.py`:

```python
# auth.py

import re
from bumbo.middleware import Middleware


STATIC_TOKEN = "ae4CMvqBe2"


class TokenMiddleware(Middleware):
    _regex = re.compile(r"^Token: (\w+)$")

    def process_request(self, req):
        header = req.headers.get("Authorization", "")
        match = self._regex.match(header)
        token = match and match.group(1) or None
        req.token = token
```

Tokens will be sent as a header `Authorization` and its value will look like this: `Token: ae4CMvqBe2`. That's why regex is used here to parse the value of the header `Authorization`. If there is a match, we attached that `token` to the request. Otherwise, if there's not a match, we gave it a value of `None`.

> Remember that the custom middlewares that you create should always inherit from the `Middleware` class that you created in your framework.

Now that requests have the `token` property, you can go ahead and write the `login_required` decorator for handlers:

```python
# auth.py

import re
from bumbo.middleware import Middleware

...

class InvalidTokenException(Exception):
    pass


def login_required(handler):
    def wrapped_view(request, response, *args, **kwargs):
        token = getattr(request, "token", None)

        if token is None or not token == STATIC_TOKEN:
            raise InvalidTokenException("Invalid Token")

        return handler(request, response, *args, **kwargs)

    return wrapped_view
```

Here, we created a custom exception class, which we'll use later when creating an exception handler. We also created a `login_required` decorator that takes a `token` from the request and then checks if it is valid. If it's valid, the handlers are called; and, if not, an `InvalidTokenException` exception is raised.

## Create and Delete Books

Since everything is ready, you can now add the `TokenMiddleware` and create a handler for creating books in *app.py*:

```python
# app.py

from bumbo.api import API

from auth import login_required, TokenMiddleware, STATIC_TOKEN
from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")
app.add_middleware(TokenMiddleware)

...

@app.route("/books", allowed_methods=["post"])
@login_required
def create_book(req, resp):
    book = book_storage.create(**req.POST)

    resp.status_code = 201
    resp.json = book._asdict()
```

So, we got the data from `req.POST`, which is a dict that looks like `{"name": "7 habits", "author": "Stephen Covey"}`, and gave it to `book_storage.create()` as params. Then, we changed the status code to 201, which indicates that the request has successfully led to the creation of a resource. Lastly, we sent the newly created book as a dictionary to the response.

While we're at it, we can create a handler for deleting a book as well:

```python
# app.py

from bumbo.api import API

from auth import login_required, TokenMiddleware, STATIC_TOKEN
from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")
app.add_middleware(TokenMiddleware)

...

@app.route("/books/{id:d}", allowed_methods=["delete"])
@login_required
def delete_book(req, resp, id):
    book_storage.delete(id)

    resp.status_code = 204
```

Here, we called the `delete()` method of the `BookStorage` class and passed the `id` of the book to delete as a param. Then, we changed the status code of the response to 204 to indicate that the resource has successfully been deleted.

The full *app.py* file should now look like this:

```python
# app.py

from bumbo.api import API

from auth import login_required, TokenMiddleware, STATIC_TOKEN
from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")
app.add_middleware(TokenMiddleware)


@app.route("/", allowed_methods=["get"])
def index(req, resp):
    books = book_storage.all()
    resp.html = app.template("index.html", context={"books": books})


@app.route("/login", allowed_methods=["post"])
def login(req, resp):
    resp.json = {"token": STATIC_TOKEN}


@app.route("/books", allowed_methods=["post"])
@login_required
def create_book(req, resp):
    book = book_storage.create(**req.POST)

    resp.status_code = 201
    resp.json = book._asdict()


@app.route("/books/{id:d}", allowed_methods=["delete"])
@login_required
def delete_book(req, resp, id):
    book_storage.delete(id)

    resp.status_code = 204
```

## Frontend

Now that we're done with the backend, let's turn out attention to the frontend.

Change the *index.html* to the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>List of Books</title>
  <link href="/static/main.css" type="text/css" rel="stylesheet">
</head>
<body>

<div id="login-form-wrapper">
  <form id="login-form" method="post">
    <label for="username">Username: </label>
    <input type="text" name="username" id="username" required>

    <label for="password">Password: </label>
    <input type="password" name="password" id="password" required>

    <button id="login-button">Log in</button>
  </form>
</div>

<br>

<table>
  <tr>
    <th>ID</th>
    <th>Name</th>
    <th>Author</th>
    <th>Actions</th>
  </tr>

  {% for book in books %}
    <tr>
      <td>{{ book.id }}</td>
      <td>{{ book.name }}</td>
      <td>{{ book.author }}</td>
      <td><button data-id="{{book.id}}" class="delete-button">Delete</button></td>
    </tr>
  {% endfor %}
</table>

<hr>

<form id="create-form" method="post">
  <label for="name">Name: </label>
  <input type="text" name="name" id="name" required>

  <label for="author">Author: </label>
  <input type="text" name="author" id="author" required>

  <button id="create-button">Create</button>
</form>

<script
  src="https://code.jquery.com/jquery-3.4.1.min.js"
  integrity="sha256-CSXorXvZcTkaix6Yvo6HppcZGetbYMGWSFlBw8HfCJo="
  crossorigin="anonymous">
</script>
<script src="/static/app.js"></script>
</body>
</html>
```

The main things to note here are that we:

- Added a login form to the top
- Added a form to create a book to the bottom
- Added jQuery as a script to the bottom
- Added `app.js` as a script to the bottom where we'll write the frontend logic of the app

At this point, the app should look like this:

![app](https://testdriven.io/static/images/courses/python-web-framework/full-frontend-look.png)

I won't bother you too much with the frontend logic since you're not here to learn how to do basic JavaScript. Thus, create an *app.js* in the "static" folder and put the following inside:

```javascript
$(document).ready(function(){

  let token = localStorage.getItem("token");

  if (token !== null) {
    $("#login-form").hide();
  }else{
    $("#login-form").show();
  }

  $("#login-button").on("click", function (event) {
    event.preventDefault();
    $.ajax({
        type: 'POST',
        url: "/login",
    }).done(function (data) {
      token = data["token"];
      localStorage.setItem("token", token);
      $("#login-form").hide();
    }).fail(function () {
      console.log("Failed to login!");
    });
  });

  $("#create-button").on("click", function (e) {
    e.preventDefault();

    let name = $("#name").val();
    let author = $("#author").val();

    $.ajax({
        type: 'POST',
        url: "/books",
        headers: {
            "Authorization":"Token: " + token
        },
        data: {"name": name, "author": author},
        statusCode: {
          401: function() {
            alert('Authorize First!');
          }
        }
    }).done(function () {
      location.reload();
    }).fail(function () {
      console.log("Failed to create!");
    })
  });

  $(".delete-button").on("click", function (e) {
    let r = confirm("Sure you want to delete this book?");
    let bookId = $(this).data("id");

    if (r === true) {
      $.ajax({
          type: 'DELETE',
          url: "/books/"+bookId,
          headers: {
              "Authorization":"Token: " + token
          },
          statusCode: {
            401: function() {
              alert('Authorize First!');
            }
          }
      }).done(function () {
        location.reload();
      }).fail(function () {
        console.log("Failed to delete!" );
      });
    } else {
      console.log("You pressed Cancel!");
    }
  })
});
```

Restart Gunicorn and test out the app.

> You can input anything for the username and password.

Everything works well if you do things in the correct order -- e.g., log in before trying to create or delete a book. However, the app will not complain if you try to create a book without logging in first, but won't create a book either. The reason is that the backend will throw an `InvalidTokenException`, but it's not being handled.

To fix, create an exception handler that catches `InvalidTokenException` and changes the status code of the response accordingly.

Add the following to *auth.py*:

```python
# auth.py

import re
from bumbo.middleware import Middleware

...

def on_exception(req, resp, exception):
    if isinstance(exception, InvalidTokenException):
        resp.text = "Token is invalid"
        resp.status_code = 401
```

*auth.py* should now look like:

```python
# auth.py

import re
from bumbo.middleware import Middleware


STATIC_TOKEN = "ae4CMvqBe2"


class TokenMiddleware(Middleware):
    _regex = re.compile(r"^Token: (\w+)$")

    def process_request(self, req):
        header = req.headers.get("Authorization", "")
        match = self._regex.match(header)
        token = match and match.group(1) or None
        req.token = token


class InvalidTokenException(Exception):
    pass


def login_required(handler):
    def wrapped_view(request, response, *args, **kwargs):
        token = getattr(request, "token", None)

        if token is None or not token == STATIC_TOKEN:
            raise InvalidTokenException("Invalid Token")

        return handler(request, response, *args, **kwargs)

    return wrapped_view


def on_exception(req, resp, exception):
    if isinstance(exception, InvalidTokenException):
        resp.text = "Token is invalid"
        resp.status_code = 401
```

Now, in *app.py* add this exception handler:

```python
# app.py

from bumbo.api import API

from auth import login_required, TokenMiddleware, STATIC_TOKEN, on_exception
from storage import BookStorage


app = API()
book_storage = BookStorage()
book_storage.create(name="7 habits of highly effective people", author="Stephen Covey")
app.add_middleware(TokenMiddleware)
app.add_exception_handler(on_exception)

...
```

Restart Gunicorn. Now, before attempting to create or delete a book, you'll need to either open a new browser window in Incognito mode or delete the `token` from LocalStorage. Once logged out, try creating or delete a book. You should see an alert telling you to authorize first. Great! Everything is working as it should.

## Conclusion

In this chapter, you developed a web app with the framework you built. Isn't that great? Sure, some of the things were mocked but they could also be added to the framework or be outsourced to a third-party library.

At this point, the web app is functional only in your localhost. In the next chapter, you'll deploy it to cloud so that users of your app are able to access it.

See you in the next chapter!

# Deploying to Heroku

##### Chapter 12

In the previous chapters, you built a framework and then used it to build a web application. Unfortunately, this application is only available for you to access since it only works on your local machine. In this chapter, to make this application available to the outside world, you'll deploy the web application to [Heroku](https://www.heroku.com/).

Heroku is a Platfrom as a Service (PaaS) that helps with deploying, managing, and scaling apps. From my experience, Heroku gives you the simplest path to delivering apps quickly. This is exactly what you are going to witness in this chapter.

## Heroku Setup

The first thing you need to do is sign up for a free [Heroku account](https://signup.heroku.com/dc) (if you don't already have one).

Next, you need to install and set up the [Heroku Command Line Interface](https://devcenter.heroku.com/articles/heroku-cli) (CLI) that you will use to deploy and scale your applications in this chapter. You'll also use it to view application logs. Refer to the [Download and install](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) guide from the official Heroku documentation for help installing the CLI tool.

> You will need [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) for the Heroku CLI to work.

Now, you need to prepare your web application for deployment. There are two things that you need to do for that.

First, create a *requirements.txt* file in the project root where you'll write the required packages and their versions. The web app you created requires only two packages -- your framework and Gunicorn. So, put the following in *requirements.txt*:

```
bumbo==0.0.2
gunicorn==19.9.0
```

> This will install the framework that I wrote. In order to install yours, change the name and the version accordingly. You can find that information by doing `pip freeze` on your command line.

Next, you need to tell Heroku what command to execute in order to start your web application. That command is `gunicorn app:app` as you have been doing thus far. And the way you tell Heroku about that is with the help of a file called *Procfile*. So, create a file called *Procfile* (without a file extension) and put the following inside:

```
web: gunicorn app:app
```

This tells Heroku that there is a process called `web` that is run by the `gunicorn app:app` command.

Next, if you haven't already done so, initialize a new git repo in the project root:

```
(venv)$ git init
```

There are a couple of files and folders that you don't want to store in our git repository and thus you need to tell git to ignore them. So, create a *.gitignore* file and put the following in it:

```
venv
__pycache__/
```

The first item is the virtual environment folder and the second is for the bytecode cache that gets automatically generated by Python.

Now, add and commit everything else in your git repo:

```
(venv)$ git add .
(venv)$ git commit -m "initial commit"
```

You are now ready to deploy the app to Heroku.

## Deployment

First, create a Heroku project by executing the following in your command line:

```
(venv)$ heroku create <project_name>
```

> Be sure to replace `<project_name>` with a unique name.

This will create a project in Heroku which you can see by [logging in to your account](https://id.heroku.com/login) and navigating to the [dashboard](https://dashboard.heroku.com/apps).

Next, push your repo to Heroku:

```
(venv)$ git push heroku master
```

If all went well, you should something similar in your console:

```
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Python app detected
remote: -----> Installing python-3.6.8
remote: -----> Installing pip
remote: -----> Installing SQLite3
remote: -----> Installing requirements with pip
remote:        Collecting bumbo==0.0.2 (from -r /tmp/build_370d47c2f9d8ea003bc958144fea4896/requirements.txt (line 1))

...

remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 45.4M
remote: -----> Launching...
remote:        Released v3
remote:        https://bumbo.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/bumbo.git
 * [new branch]      master -> master
```

And lastly, you need to ensure that at least one instance of your app is running:

```
(venv)$ heroku ps:scale web=1
```

You should see the following in the console:

```
Scaling dynos... done, now running web at 1:Free
```

That's it. Heroku will provide a domain that you can use to access your application, which will be in the format of `https://<project_name>.herokuapp.com`. Alternatively, you can run `heroku open` to open the app in your default browser.

Voila! You have deployed your web application.

If you want to access the logs of your app, run `heroku logs --tail`. You should see something like:

```
2019-08-01T13:26:28.123899+00:00 app[api]: Release v1 created by user jrahmonov2@gmail.com
2019-08-01T13:26:28.334317+00:00 app[api]: Enable Logplex by user jrahmonov2@gmail.com
2019-08-01T13:26:28.123899+00:00 app[api]: Initial release by user jrahmonov2@gmail.com
2019-08-01T13:26:28.334317+00:00 app[api]: Release v2 created by user jrahmonov2@gmail.com
2019-08-01T13:32:23.000000+00:00 app[api]: Build started by user jrahmonov2@gmail.com
2019-08-01T13:32:51.932082+00:00 app[api]: Deploy f963673f by user jrahmonov2@gmail.com
2019-08-01T13:32:51.932082+00:00 app[api]: Release v3 created by user jrahmonov2@gmail.com
2019-08-01T13:32:51.951162+00:00 app[api]: Scaled to web@1:Free by user jrahmonov2@gmail.com
2019-08-01T13:32:55.987023+00:00 heroku[web.1]: Starting process with command `gunicorn app:app`
2019-08-01T13:32:59.178850+00:00 app[web.1]: [2019-08-01 13:32:59 +0000] [4] [INFO] Starting gunicorn 19.9.0
2019-08-01T13:32:59.180164+00:00 app[web.1]: [2019-08-01 13:32:59 +0000] [4] [INFO] Listening at: http://0.0.0.0:53414 (4)
2019-08-01T13:32:59.180406+00:00 app[web.1]: [2019-08-01 13:32:59 +0000] [4] [INFO] Using worker: sync
2019-08-01T13:32:59.189183+00:00 app[web.1]: [2019-08-01 13:32:59 +0000] [10] [INFO] Booting worker with pid: 10
2019-08-01T13:32:59.231127+00:00 app[web.1]: [2019-08-01 13:32:59 +0000] [11] [INFO] Booting worker with pid: 11
2019-08-01T13:33:00.496232+00:00 heroku[web.1]: State changed from starting to up
2019-08-01T13:33:00.000000+00:00 app[api]: Build succeeded
```

## Conclusion

Congratulations! With this chapter finished, you have closed the cycle of a WSGI framework. You built a web framework, which you then used to build a web app. You then deployed it to Heroku so that others can access it.