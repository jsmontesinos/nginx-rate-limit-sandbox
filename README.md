# NGINX Rate Limit reverse proxy, Burst and nodelay sandbox

NGINX Rate limiting is more traffic-shaping than pure rate-limiting. And this is an important point to understand how it works with the `burst` and `no_delay` settings.


Tools: Docker, and [Siege](https://www.joedog.org/siege-home/) (you can brew-it, or use other cli load-testing tools like ab or artillery or anything you like!)


The Nginx config defines a few locations to test the various combinations of:
- limit_req_zone by uri or by ip
- using the burst argument (set to 5 in this case) or not
- adding nodelay to control how to deal with request going over-quota during bursts.

The rates defined are:
- 30 req/min
- burst locations allow a burst of 5

With the leaky bucket, that means a new request should be allowed ever 2 seconds.

The endpoints defined are:
- http://127.0.0.1:80/by-uri/node_express
- http://127.0.0.1:80/by-ip/node_express
- http://127.0.0.1:80/by-ip/node_express/burst0_nodelay
- http://127.0.0.1:80/by-ip/node_express/burst5
- http://127.0.0.1:80/by-ip/node_express/burst5_nodelay

The bench work has been modified to be configured as a ngnix reverse proxy plus a node express application

## Test it!

```bash
docker-compose up
```

Using Siege to send 10 concurrent requests at once on the various endpoints
The most interesting ones are the burst5 and burst5_nodelay which let you really visualize and remember how nginx deal with burst settings!

    siege -b -r 1 -c 10 -v http://127.0.0.1:80/by-ip/node_express
    siege -b -r 1 -c 10 -v http://127.0.0.1:80/by-ip/node_express/burst0_nodelay
    siege -b -r 1 -c 10 -v http://127.0.0.1:80/by-ip/node_express/burst5
    siege -b -r 1 -c 10 -v http://127.0.0.1:80/by-ip/node_express/burst5_nodelay

When doing these tests, you will want to pay attention to:
- the success/status code (obviously)
- the response time it took, both for the rate-limited requests _and_ the succesfull requests
