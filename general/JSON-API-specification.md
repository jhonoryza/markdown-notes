# What is an API?

To best describe what an API is, let’s imagine that our application or service
is like a restaurant. The frontend of the application is where you sit at the
table and eat, and the backend is the kitchen, where they prepare food for you.
Here, an API plays the role of the menu, where you can pick what you want the
kitchen to cook for you. In programming terms, an API makes it possible for a
frontend developer to request a specific task or resource from the backend.

If you look at a menu, you might notice that it consists of a finite
predetermined set of dishes: a collection of dishes that the kitchen knows how
to cook and prepare for you. The same goes for an API, but instead of having
dishes to choose from, you instead select between endpoints. Through endpoints,
we can order our backend to prepare and send back some data for us. It could be
a list of books for a bookstore or the latest comments for a blog post,
depending on the service or application that serves a solution to a problem.

# What is REST?

REST stands for Representational State Transfer and is an architectural style
used for communication between a server and a client. REST uses the HTTP or
Hypertext Transfer Protocol, as a base for the communication. We already use
HTTP for transferring HTML and other media from servers to our clients. This is,
for instance, what happens when you visit a website, so by building on an
already familiar technology, REST is easily adaptable.

REST sends data using either XML or JSON. Both of these languages are meant for
transferring data, but also meant to be readable by humans, which also makes
error tracking in REST a lot easier. REST is platform and language independent,
as long as you adapt to HTTP, you adapt to REST. Already established features of
HTTP, like SSL encryption, make it possible to transfer encrypted data across
from server to client, so there are a lot of advantages we get for free.

A disadvantage is that REST is not stateful, meaning that the state is not
carried along from one request to the other. Therefore, you always have to send
some kind of context to the server for it to know what to deliver to you. This
limitation stems from HTTP itself, so this is most likely something you are
already familiar with.

Like HTTP, REST works through requests, where you “pull” data from the server.
There is no way of “pushing” data through REST, although it is possible for the
server to do server pushing, using the HTTP 2 standard, but even that is only
initiated by a request.

Since REST relies so much on HTTP, there are some things we have to examine to
understand REST and the way communication takes place. For instance, HTTP
methods, also called verbs, play a significant role in the intention for a REST
request, as much as the HTTP Status code plays a significant role in the
answers. These are the HTTP building blocks that make up most of the base of
REST communication. Let’s take a closer look at HTTP Verbs.

**HTTP verbs**

As mentioned earlier, HTTP verbs play a significant role in the intention of a
request. The HTTP verb tells the server about how we, on the client side, intend
to handle our data. How to handle data is often looked at from a CRUD
perspective, which stands for: Create, Read, Update, Delete

**GET**

A request with a GET verb is for reading data.

**POST**

A request with a POST verb is for creating a resource.

**PUT and PATCH**

Both the PUT and the PATCH request is for updating or modifying data, but the
way they are intended is a bit different.

PUT verbs are used when all data for a resource is completely replaced with the
data given by the client.

PATCH verbs are used for a partial update or modification, instead of replacing
everything in the resource.

**DELETE**

A request with a DELETE verb is, much as the name implies, for deleting a
resource

**Status Codes**

Status codes are used by the server to tell the client whether a request has
been successfully completed. The areas that status codes cover are divided into
5 groups. Let’s take a look at a few from each group that we will be using the
most:

**2XX as Success**

The 2XX range are statuses that tell the client a request was successful. You
would think that only one status was needed here, but in some cases where, for
example, you create something, it would be nice to know if the resource has been
created.

**200 OK**

The 200 status code, which tells the client that the entire request was
successful, is the most common.

**201 Created**

This status code tells the client that one or more resources have been created.

**204 No Content**

This status code tells the client that the request has been fulfilled, but also
that there is no payload in the response.

**3XX as Redirection** The 3XX range are all statuses that tell the client about
redirections.

**301 Moved Permanently**

The 301 status code tells the client that an endpoint has been moved and should
give a new location in its payload for the client to save for future reference.
A thing to note here is that the 301 status code makes it possible to change the
HTTP verb for the request.

**307 Temporary Redirect**

The 307 status code is used for a temporary redirect. Here, the server should
give a new location in its payload, for the client to redirect to. The client
will not save any information about the redirect and will merely follow the
location given by the server and gladly hit the old endpoint time after time,
since the redirect is only temporary. A thing to note, as mentioned in the 301
status code, is that the 307 status code does not let you change the HTTP verb
in the redirect. When redirecting the user, the endpoint to which you are
redirected must match the HTTP verb from which you were redirected.

**308 Permanent Redirect**

The 308 status code is used for a permanent redirect, much like the 301
redirects. The only difference is that, like 307, it does not allow for the HTTP
verb to change from the original endpoint to the endpoint redirected to.

**4XX as Client errors**

The 4XX range are all statuses that deal with client error, which could be a
request that the client is not authenticated to do or even a request to a
misspelled endpoint, which the server cannot fulfill.

**400 Bad Request**

The 400, much like the 200, is a broad status code. All it does is tell the
client that the server could not or would not process the request. It does not
specify any reasons, and the client would have to look at the payload for
further information.

**401 Unauthorized**

The 401 status tells the client that the request could not be fulfilled due to
lacking authentication credentials.

**403 Forbidden**

The 403 status tells the client that the request could not be fulfilled due to
lacking authorization. For example, this status code could be sent back if one
user tries to access or update another user’s data. The user might be
authenticated to access the endpoint, but not have authorization to access the
data.

**404 Not Found**

The 404 status tells the client that the requested resource could not be found.
This is a pretty common status that most people, even non-developers, have been
met by.

**405 Method Not Allowed**

As we have explained earlier, HTTP methods are also called HTTP verbs in REST.
The 405 status tells the client that the request has been made with an HTTP verb
that is not allowed. This could, for instance, be a request made using a GET
verb to an endpoint that only supports the POST verb. In this case, a 405 status
should be sent to the client.

**422 Unprocessable Entity**

The official RFC4918 states that this status is to be used when the server
understands the content of the request and the syntax of the request is correct,
but was unable to process the request. An RFC stands

for Request For Comments, which can be viewed as the rules for standardizing the
internet. The number references the document in which the request has been
documented. In Laravel, the 422 status code is used for validation errors when
using REST and JSON. The JSON:API documentation says that this status is to be
used when creating or updating a resource where an attribute is invalid. Based
on all these examples, we can safely say that the 422 status code tells the
client about invalid data sent in the request.

**5XX as Server errors**

The 5XX range are all statuses that deal with the server, where it is aware that
an error has occurred or might otherwise be unable to handle the request.

**500 Internal Server Error**

This status is used as a generic error message given when it is not possible to
provide a more specific status code.

**501 Not Implemented**

This status is used when the server does not know how to fulfill the request,
but implies that it might be available in the future. This could be a new
feature that is under development.

**502 Bad Gateway**

This status is used when the server is acting as a gateway and has received an
invalid response.

**503 Service Unavailable**

This status code is used if the server is down for maintenance

**504 Gateway Timeout**

This status is used when the server is acting as a gateway and did not receive a
response within a given period.

## Top-level

Here, the JSON:API specification states that there must be a JSON object at the
root of the document, representing the top-level.

In the top-level of the document, there must be at least one of the following
members:

- data - which is the most important member that contains the primary data of
  the document.
- included - which is a member that contains all resource objects that are
  related to the primary data and/or related to each other. We will touch more
  on this when we get to the section about resource objects and relationships.
- errors - which is a member that contains all error objects.
- jsonapi - which is a member that contains the server’s implementation of the
  JSON:API specification
- meta - which is a member that contains all non-standard meta information.

### Data

When requesting a single resource like this:

```
GET: /books/1
```

the data member of the returned document should be structured like this: (note
that we are omitting the attributes and relationships for the sake of
simplicity)

```json
{
 “data”: {
   “id”: “1”,
   “type”: “books”,
   “attributes”: {
     “title”: “Build an API with Laravel”,
     “publication_year”: “2019”
   }
 }
}
```

#### Relationships

Unlike the attributes member, where you are in charge of the members, the
relationships have a more strict set of rules. A relationship member must
contain at least one of the following members:

- links
  - self
  - related
- data
- meta

```json
{
 “data”: {
   “id”: “1”,
   “type”: “books”,
   “attributes”: {
     “title”: “Build an API with Laravel”,
     “publication_year”: “2019”
   },
   “relationships”: {
     “author”: {
       “links”: {
         “self”: “http://example.com/books/1/relationships/authors”,
         “related”: “http://example.com/books/1/authors”
       },
       “data”: {
         “id”: “5”,
         “type”: “authors”
       }
     },
     “comments”: {
       “links”: {
         “self”: “http://example.com/books/1/relationships/comments”,
         “related”: “http://example.com/books/1/comments”
       },
       “data”: [
         {
           “id”: “16”,
           “type”: “comments”
         },
         {
           “id”: “28”,
           “type”: “comments”
         }
       ]
		 }
	 }
 }
}
```

#### Sorting

Sorting data is a great feature to have in an API. Think about sorting just like
ORDER BY in your database. You get the ability to sort your data based on member
names in a more dynamic way, instead of being limited by the way the API
developer may have thought was the best way to sort the data.

The value of the parameter is the member name of the attribute you want to sort
by. It would look something like this:

```
GET: /books?sort=title
```

If you want to support multiple sort fields, these should be separated by a
comma like this:

```
GET: /books?sort=title, publication_date
```

When ordering a database query by a column name, we are able to tell if the
ordering should be done in ascending order or descending order. The same thing
goes for sorting a collection, according to the JSON:API specification. Here, a
sorting is always done in ascending order unless you prefix a sort field with a
minus, in which case it will be sorted in descending order. It would look
something like this:

```
GET: /authors?sort=-age
```

Here, you will get the oldest authors first, descending until the youngest
author in the collection

#### Pagination

The way pagination is done in the JSON:API specification is through a links
object in the root of the response document. The links object must have the
following members used for pagination links:

- first - which is the first page of data
- last - which is the last page of data
- prev - which is the previous page of data
- next - which is the next page of data

The links object in the document would look something like this:

```json
{
 “data”: [
   {
     “id”: “4”,
     “type”: “books”,
     “attributes”: {
       “title”: “Lorem Ipsum”
     }
   },
   {
     “id”: “5”,
     “type”: “books”,
     “attributes”: {
       “title”: “Lorem Ipsum”
     }
   },
   {
     “id”: “6”,
     “type”: “books”,
     “attributes”: {
       “title”: “Lorem Ipsum”
     }
   },
 ],
 “links”: {
   “first”: “http://example.com/books?page=1”,
   “last”: “http://example.com/books?page=5”,
   “prev”: “http://example.com/books?page=1”,
   “next”: “http://example.com/books?page=3”
 }
}
```

### Included

When building an API or an application for that matter, you have to make some
thoughts about optimization and make sure your application performs as intended.
One optimization could be to reduce the number of HTTP requests as much as
possible.

One way to do this is to use the included member. The reason for this is that it
makes it possible for you to include the related resources of the fetched
resource, which will then be the resources defined in the data member in the
relationships object.

Instead of having to make a new request for the related resources, they can just
be included in the current response.

In this case, the resource objects sent in the included member will correspond
to the resource identifier objects given in the relationships’ data member.

Let’s build upon the previous examples to give a better idea of this:

```json
{
 “data”: {
   “id”: “1”,
   “type”: “books”,
   “attributes”: {
     “title”: “Build an API with Laravel”,
     “publication_year”: “2019”
   }
 },
 “included”: [
   {
     “id”: “5”,
     “type”: “authors”,
     “attributes”: {
       “name”: “Wacky Studio”
     }
   },
   {
     “id”: “16”,
     “type”: “comments”,
     “attributes”: {
       “body”: “Hello world”
     }
   },
   {
     “id”: “28”,
     “type”: “comments”,
     “attributes”: {
       “body”: “Foo bar”
     }
   }
 ]
}
```

it should be possible to specify which relationships that should be included in
the response using a comma separated list:

```
GET: /books/1?include=authors,comments
```

It should be possible to request resources related to other resources using a
dot-separated path for each relationship name. In our example, a book has a
relationship with authors and comments, but each comment also has an author, in
the form of a user who created the comment. If we wanted to include the users
for each comment, it should be possible to do this by adding the related
resource like this:

```
GET: /books/1?include=authors,comments.users
```

If it isn’t possible to fetch the related resource, you should return with a 400
Bad Request.

### Errors

It’s time to take a look at errors. Let’s face it, no matter how great of an
application you build, errors will always be a part of it. Some are intentional,
like validation rules that are not met, and some are not. In both cases, it’s
crucial that they are handled in a consistent way, so both you and the consumer
of your API know what went wrong.

```json
{
 “errors”: [
   {
     “title”: “Validation error”,
     “source”: {
       “pointer”: “/data/attributes/name”
     },
     “detail”: “The name field is required.”
   },
   {
     “title”: “Validation error”,
     “source”: {
       “pointer”: “/data/attributes/email”
     },
     “detail”: “The email must be a valid email address.”
   },
   {
     “title”: “Validation error”,
     “source”: {
       “pointer”: “/data/attributes/age”
     },
     “detail”: “The :attribute must be a number.”
   }
 ]
}
```
