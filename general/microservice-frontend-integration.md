So now we have a bunch of services running on a host on different ports. What
about the frontend? How the frontend will know that when a user clicks on the
“Create Order” button we need to send a request to localhost:8001/api/v1/posts,
but when a user clicks on the “Create Product” button we need to use
localhost:8002 because this is where the Product service listens?

The answer is that our frontend does not know anything about these different
services and ports.

We need a gateway between the FE and the services. Something like this

![](https://minio.labkita.my.id/laravelblog/Screenshot%202024-08-05%20at%209.58.08%20PM.png)
So the gateway sits between the FE and the BE services. It acts as a reverse
proxy. It takes requests from the FE, maps them to the right service, and sends
the request to it.

You can implement an API gateway in (at least) two ways:

- Reverse proxy. You can use Nginx or haproxy. It’s very simple and takes about
  5 lines of config by service. It only proxies your requests.
- You can write your own gateway “service.” It acts exactly like an Nginx
  reverse proxy, but it actually contains some code, so you can do extra tasks
  like:
  - Authenticate every request via an Auth service. And by the way, in the
    microservices world, you want to use stateless, jwt token-based
    authentication.
  - Serving the responses from the cache. If you have a gateway service, you can
    cache in one place, and you don’t have to implement caching in every
    service. I’m talking about Redis or Memcached.

Each solution can work, but if you choose the second one, please don’t use
Laravel. Don’t get me wrong, it’s an excellent framework, but it’s a behemoth
for this job. It will be really slow compared to a Nodejs express or a Python
flask gateway
