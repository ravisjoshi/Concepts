## nginx
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#
----
----
## HTTP Load Balancing
### Proxying HTTP Traffic to a Group of Servers
To start using NGINX Plus or NGINX Open Source to load balance HTTP traffic to a group of servers, first you need to define the group with the `upstream` directive. The directive is placed in the `http` context.

Servers in the group are configured using the `server` directive (not to be confused with the server block that defines a virtual server running on NGINX). For example, the following configuration defines a group named *backend* and consists of three server configurations (which may resolve in more than three actual servers):
```
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}
```
To pass requests to a server group, the name of the group is specified in the `proxy_pass` directive (or the `fastcgi_pass`, `memcached_pass`, `scgi_pass`, or `uwsgi_pass` directives for those protocols.) In the next example, a virtual server running on NGINX passes all requests to the *backend* upstream group defined in the previous example:
```
server {
    location / {
        proxy_pass http://backend;
    }
}
```
The following example combines the two snippets above and shows how to proxy HTTP requests to the *backend* server group. The group consists of three servers, two of them running instances of the same application while the third is a backup server. Because no load‑balancing algorithm is specified in the `upstream` block, NGINX uses the default algorithm, `Round Robin`:
```
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Choosing a Load-Balancing Method

NGINX Open Source supports four load‑balancing methods, and NGINX Plus adds two more methods:

* *Round Robin* – Requests are distributed evenly across the servers, with `server weights` taken into consideration. This method is used by default (there is no directive for enabling it):
```
    upstream backend {
       # no load balancing method is specified for Round Robin
       server backend1.example.com;
       server backend2.example.com;
    }
```
* *Least Connections* – A request is sent to the server with the least number of active connections, again with `server weights` taken into consideration:
```
    upstream backend {
        least_conn;
        server backend1.example.com;
        server backend2.example.com;
    }
```
* *IP Hash* – The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.
```
    upstream backend {
        ip_hash;
        server backend1.example.com;
        server backend2.example.com;
    }
```
If one of the servers needs to be temporarily removed from the load‑balancing rotation, it can be marked with the `down` parameter in order to preserve the current hashing of client IP addresses. Requests that were to be processed by this server are automatically sent to the next server in the group:
```
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com down;
    }
```
* *Generic Hash* – The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination. For example, the key may be a paired source IP address and port, or a URI as in this example:
```
    upstream backend {
        hash $request_uri consistent;
        server backend1.example.com;
        server backend2.example.com;
    }
```
The optional `consistent` parameter to the hash directive enables `ketama` consistent‑hash load balancing. Requests are evenly distributed across all upstream servers based on the user‑defined hashed key value. If an upstream server is added to or removed from an upstream group, only a few keys are remapped which minimizes cache misses in the case of load‑balancing cache servers or other applications that accumulate state.

Least Time (NGINX Plus only) – For each request, NGINX Plus selects the server with the lowest average latency and the lowest number of active connections, where the lowest average latency is calculated based on which of the following `parameters` to the least_time directive is included:
    * `header` – Time to receive the first byte from the server
    * `last_byte` – Time to receive the full response from the server
    * `last_byte` inflight – Time to receive the full response from the server, taking into account incomplete requests
```
    upstream backend {
        least_time header;
        server backend1.example.com;
        server backend2.example.com;
    }
```
* *Random* – Each request will be passed to a randomly selected server. If the two parameter is specified, first, NGINX randomly selects two servers taking into account server weights, and then chooses one of these servers using the specified method:
        `least_conn` – The least number of active connections
        `least_time=header` (NGINX Plus) – The least average time to receive the response header from the server ($upstream_header_time)
        `least_time=last_byte` (NGINX Plus) – The least average time to receive the full response from the server ($upstream_response_time)
```
    upstream backend {
        random two least_time=last_byte;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        server backend4.example.com;
    }
```
The *Random* load balancing method should be used for distributed environments where multiple load balancers are passing requests to the same set of backends. For environments where the load balancer has a full view of all requests, use other load balancing methods, such as round robin, least connections and least time.

*Note:* When configuring any method other than Round Robin, put the corresponding directive (hash, ip_hash, least_conn, least_time, or random) above the list of server directives in the upstream {} block.

### Server Weights
By default, NGINX distributes requests among the servers in the group according to their weights using the Round Robin method. The `weight` parameter to the `server` directive sets the weight of a server; the default is 1:
```
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
In the example, *backend1.example.com* has weight `5`; the other two servers have the default weight (`1`), but the one with IP address `192.0.0.1` is marked as a backup server and does not receive requests unless both of the other servers are unavailable. With this configuration of weights, out of every `6` requests, `5` are sent to *backend1.example.com* and `1` to *backend2.example.com*.

### Server Slow-Start
The server slow‑start feature prevents a recently recovered server from being overwhelmed by connections, which may time out and cause the server to be marked as failed again.

In NGINX Plus, slow‑start allows an upstream server to gradually recover its weight from 0 to its nominal value after it has been recovered or became available. This can be done with the `slow_start` parameter to the `server` directive:
```
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
The time value (here, `30` seconds) sets the time during which NGINX Plus ramps up the number of connections to the server to the full value.

Note that if there is only a single server in a group, the `max_fails`, `fail_timeout`, and `slow_start` parameters to the server directive are ignored and the server is never considered unavailable.


### Enabling Session Persistence
Session persistence means that NGINX Plus identifies user sessions and routes all requests in a given session to the same upstream server.

NGINX Plus supports three session persistence methods. The methods are set with the `sticky` directive. (For session persistence with NGINX Open Source, use the `hash` or `ip_hash` directive as described `above`.)

* *Sticky cookie* – NGINX Plus adds a session cookie to the first response from the upstream group and identifies the server that sent the response. The client’s next request contains the cookie value and NGINX Plus route the request to the upstream server that responded to the first request:
```
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        sticky cookie srv_id expires=1h domain=.example.com path=/;
    }
```
    In the example, the `srv_id` parameter sets the name of the cookie. The optional `expires` parameter sets the time for the browser to keep the cookie (here, `1` hour). The optional domain parameter defines the `domain` for which the cookie is set, and the optional `path` parameter defines the path for which the cookie is set. This is the simplest session persistence method.

* *Sticky route* – NGINX Plus assigns a “route” to the client when it receives the first request. All subsequent requests are compared to the `route` parameter of the `server` directive to identify the server to which the request is proxied. The route information is taken from either a cookie or the request URI.
```
    upstream backend {
        server backend1.example.com route=a;
        server backend2.example.com route=b;
        sticky route $route_cookie $route_uri;
    }
```
* *Sticky learn* method – NGINX Plus first finds session identifiers by inspecting requests and responses. Then NGINX Plus “learns” which upstream server corresponds to which session identifier. Generally, these identifiers are passed in a HTTP cookie. If a request contains a session identifier already “learned”, NGINX Plus forwards the request to the corresponding server:
```
    upstream backend {
       server backend1.example.com;
       server backend2.example.com;
       sticky learn
           create=$upstream_cookie_examplecookie
           lookup=$cookie_examplecookie
           zone=client_sessions:1m
           timeout=1h;
    }
```
In the example, one of the upstream servers creates a session by setting the cookie `EXAMPLECOOKIE` in the response.
The mandatory `create` parameter specifies a variable that indicates how a new session is created. In the example, new sessions are created from the cookie `EXAMPLECOOKIE` sent by the upstream server.

The mandatory `lookup` parameter specifies how to search for existing sessions. In our example, existing sessions are searched in the cookie `EXAMPLECOOKIE` sent by the client.

The mandatory `zone` parameter specifies a shared memory zone where all information about sticky sessions is kept. In our example, the zone is named *client_sessions* and is `1` megabyte in size.

This is a more sophisticated session persistence method than the previous two as it does not require keeping any cookies on the client side: all info is kept server‑side in the shared memory zone.

If there are several NGINX instances in a cluster that use the “sticky learn” method, it is possible to sync the contents of their shared memory zones on conditions that:
    * the zones have the same name
    * the `zone_sync` functionality is configured on each instance
    * the sync parameter is specified
```
       sticky learn
           create=$upstream_cookie_examplecookie
           lookup=$cookie_examplecookie
           zone=client_sessions:1m
           timeout=1h
           sync;
    }
```






----
----




----
----



.
