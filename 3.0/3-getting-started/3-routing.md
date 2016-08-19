---
title: Routing
permalink: routing
description: Routes in AdonisJs
categories:
- getting-started
---

{{TOC}}

Routes are defined inside `app/Http/routes.js` file and loaded automatically when you start the HTTP server. It is a good practice to keep this file clean and move all required logic to different files/directories.

## Basic Example

In real world, it is good practice to attach Controller methods to your routes. But for the most basic route, you can make use of a `Closure`.

```javascript
// importing the route provider
const Route = use('Route')

Route.get('/about', function * (request, response) {
  response.send('You are looking at the about page')
})
```

Above we defined a route for the URL `/about`. Whenever someone will visit this URL, they will see the message **You are looking at the about page**.

## HTTP Verbs

HTTP verbs are also known as HTTP methods. `Route.get` defines a route for the `GET` method.

Below is the list of Route methods and their corresponding HTTP verbs.

Verb |    Method
-----|-------
GET    | Route.get
POST |    Route.post
PUT |    Route.put
PATCH    | Route.patch
DELETE |    Route.delete

Above are the commonly used HTTP verbs, for uncommon verbs you can make use of the `Route.route` method.

#### route(url, verbs, action)
```javascript
const Route = use('Route')

Route.route('/', 'COPY', function * (request, response) {
  // Your code
})

// or

Route.route('/', ['COPY', 'MOVE'], function * (request, response) {
  // Your code
})
```

## Multiple HTTP Verbs

AdonisJs router is not only limited to a single verb at a time. Make use of the below methods, if you want to respond to multiple verbs using the same handler.

#### match(verbs, url, action)
```javascript
Route.match(['GET', 'POST'], '/', function * (request, response) {
  // Your code
})
```

#### any(url, action)
```javascript
Route.any('/', function * (request, response) {
  // Your code
})
```

## Route Parameters

Route parameters are dynamic URL segments. Which means you can register a route to accept the user id within the URL.

```javascript
Route.get('/user/:id', function * (request, response) {
  const userId = request.param('id')
  response.send(`Profile for user with id ${userId}`)
})
```

Now visiting http://localhost:3333/user/1 will return **Profile for user with id 1**. Here `:id` is a parameter and in order to get the value you make use of `request.param()` method.

In order to make params optional, you need to append `?` to the param name.

```javascript
Route.get('/user/:id?', function * (request, response) {
  const userId = request.param('id', 1)
  response.send(`Profile for user with id ${userId}`)
})
```

#### request.param(key, [defaultValue])

```javascript
const id = request.param('id')
```


#### request.params()

To get all dynamic segments as an object for a given url.

```javascript
const params = request.params()
```

## Route Formats

In order to build data agnostic applications, it is very important to be explicit with your URLs. Route formats is a step towards that goal.

#### formats(values, [strict=false])

`formats` method will define a route with multiple extensions. This is helpful when you have an API, that can return both JSON and XML.

```javascript
Route
  .get('/users', 'UserController.index')
  .formats(['json', 'xml'])
```

You can identify the format of a request inside your route action.

```javascript
Route
  .get('/users', function * (request, response) {
    const format = request.format()

    if (format === 'json') {
      response.json(users)
    } else {
      yield response.sendView('users', {users})
    }
  })
  .formats(['json'])
```

Setting `strict=true` will not register the default route. It will only register the route will defined formats.

## Named Routes

Routes are defined inside `app/Http/routes.js` file, but they are used everywhere. For example:

1. Inside a view, to create navigation bar.
2. Inside Controllers, to redirect to a different URL, etc.

As your application will grow, you will find yourself changing route URLs quite often.

Now changing them inside the routes file is quite easy, but finding their references inside the views and controllers is not something you are going to enjoy.

#### as(name)

`as()` method will give a unique name/identity to a route. So that you can use it with its name instead of the URL.

```javascript
Route
  .get('/user/:id', 'UserController.show')
  .as('profile')
```

Now you can use it inside your views as using the linkTo helper.

#### linkTo(name, text, [data], [target])

```twig
{{ linkTo('profile', 'View Profile', { id: 1 }) }}
{{ linkTo('profile', 'View Profile', { id: 1 } , '_blank') }}

{# returns #}
<a href="/user/1"> View Profile </a>
<a href="/user/1" target="_blank"> View Profile </a>
```

#### route([data])
The route is a filter which can be used when you just want the link to a route, instead of a complete anchor tag.

```twig
<a href="{{ 'profile' | route({ id: 1 }) }}"> View Profile </a>
```


## Prefixing Routes

You make use of Route group in order to define a prefix for a bunch of routes.

```javascript
Route.group('v1', function () {
  Route.get('/users', '...')
}).prefix('/api/v1')
```

Above route will have an URL `/api/v1/users`.

## Domains

AdonisJs makes it super easy to define routes only for a given domain.

```javascript
Route.group('api', function () {
  Route.get('/users', '...')
}).domain('api.example.org')
```

`/users` will only work if the base url of the request is `api.example.org`.

Let's take a step forward and define dynamic domains. Dynamic domains are defined in the same way as request parameters.

```javascript
Route.group('v1', function () {
  Route.get('/account', function * (request, response) {
    const user = request.param('user')
  })
}).domain(':user.example.org')
```

This way you can have subdomains for each user. For example, **virk.example.org** or **joe.example.org**.

## Route Renderer

Route renderer is a step towards eliminating the dead code. Every application has a requirement to render some dumb views.

Now we call them **dumb views**, because there is no logical processing required to render these views. For example:

1. An about page.
2. A welcome screen.
3. Contact page to display a form.

In order to render above views, all you need is to send html back to the browser. Same can be done using the `on` and `render` method.

```javascript
Route.on('/about').render('about')
Route.on('/contact').render('contact')
Route.on('/').render('welcome')
```

## Form Method Spoofing

We learned about different HTTP verbs, but unfortunately standard HTML forms do not allow anything over GET and POST.

Form method spoofing makes it easier to define the verb as a query string.

```twig
<form method="POST" action="/create?_method=PUT">
  <!-- Your Form -->
</form>
```

`_method` takes precedence over the actual HTTP verb, so in above case, AdonisJs will call the route action defined with PUT verb.

You need to allow method spoofing inside the configuration file for it to work.

##### config/app.js
```javascript
http: {
  allowMethodSpoofing: true
}
```
