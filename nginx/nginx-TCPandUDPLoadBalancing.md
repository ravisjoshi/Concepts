## nginx
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#
----
### Prerequisites
* Latest NGINX Plus (no extra build steps required) or latest NGINX Open Source built with the --with-stream configuration flag
* An application, database, or service that communicates over TCP or UDP
* Upstream servers, each running the same instance of the application, database, or service

### Configuring Reverse Proxy
First, you will need to configure reverse proxy so that NGINX Plus or NGINX Open Source can forward TCP connections or UDP datagrams from clients to an upstream group or a proxied server.

Open the NGINX configuration file and perform the following steps:

1. Create a top‑level `stream {}` block:
```
    stream {
        # ...
    }
```
2. Define one or more `server {}` configuration blocks for each virtual server in the top‑level `stream {}` context.

3. Within the `server {}` configuration block for each server, include the listen directive to define the IP address and/or port on which the server listens.

For UDP traffic, also include the `udp` parameter. As TCP is the default protocol for the `stream` context, there is no tcp parameter to the `listen` directive:
```
    stream {

        server {
            listen 12345;
            # ...
        }

        server {
            listen 53 udp;
            # ...
        }
        # ...
    }
```
4. Include the proxy_pass directive to define the proxied server or an upstream group to which the server forwards traffic:
```
    stream {

        server {
            listen     12345;
            #TCP traffic will be forwarded to the "stream_backend" upstream group
            proxy_pass stream_backend;
        }

        server {
            listen     12346;
            #TCP traffic will be forwarded to the specified server
            proxy_pass backend.example.com:12346;
        }

        server {
            listen     53 udp;
            #UDP traffic will be forwarded to the "dns_servers" upstream group
            proxy_pass dns_servers;
        }
        # ...
    }
```
5. If the proxy server has several network interfaces, you can optionally configure NGINX to use a particular source IP address when connecting to an upstream server. This may be useful if a proxied server behind NGINX is configured to accept connections from particular IP networks or IP address ranges.

Include the `proxy_bind` directive and the IP address of the appropriate network interface:
```
    stream {
        # ...
        server {
            listen     127.0.0.1:12345;
            proxy_pass backend.example.com:12345;
            proxy_bind 127.0.0.1:12345;
        }
    }
```
6. Optionally, you can tune the size of two in‑memory buffers where NGINX can put data from both the client and upstream connections. If there is a small volume of data, the buffers can be reduced which may save memory resources. If there is a large volume of data, the buffer size can be increased to reduce the number of socket read/write operations. As soon as data is received on one connection, NGINX reads it and forwards it over the other connection. The buffers are controlled with the `proxy_buffer_size` directive:
```
    stream {
        # ...
        server {
            listen            127.0.0.1:12345;
            proxy_pass        backend.example.com:12345;
            proxy_buffer_size 16k;
        }
    }
```
----
### Configuring TCP or UDP Load Balancing
To configure load balancing:

1. Create a group of servers, or an upstream group whose traffic will be load balanced. Define one or more `upstream {}` configuration blocks in the top‑level `stream {}` context and set the name for the upstream group, for example, `stream_backend` for TCP servers and `dns_servers` for UDP servers:
```
    stream {

        upstream stream_backend {
            # ...
        }

        upstream dns_servers {
            # ...
        }

        # ...
    }
```
Make sure that the name of the upstream group is referenced by a `proxy_pass` directive, like those configured above for reverse proxy.

2. Populate the upstream group with upstream servers. Within the `upstream {}` block, add a `server` directive for each upstream server, specifying its IP address or hostname (which can resolve to multiple IP addresses) and an obligatory port number. Note that you do not define the protocol for each server, because that is defined for the entire upstream group by the parameter you include on the `listen` directive in the `server` block, which you have created earlier.
```
    stream {

        upstream stream_backend {
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12346;
            # ...
        }

        upstream dns_servers {
            server 192.168.136.130:53;
            server 192.168.136.131:53;
            # ...
        }

        # ...
    }
```
3. Configure the load‑balancing method used by the upstream group. You can specify one of the following methods:

    * *Round Robin* – By default, NGINX uses the Round Robin algorithm to load balance traffic, directing it sequentially to the servers in the configured upstream group. Because it is the default method, there is no round‑robin directive; simply create an `upstream {}` configuration block in the top‑level `stream {}` context and add server directives as described in the previous step.

    * *Least Connections* – NGINX selects the server with the smaller number of current active connections.

    * *Least Time* (NGINX Plus only) – NGINX Plus selects the server with the lowest average latency and the least number of active connections. The method used to calculate lowest average latency depends on which of the following parameters is included on the `least_time` directive:
        * `connect`  – Time to connect to the upstream server
        * `first_byte` – Time to receive the first byte of data
        * `last_byte`  – Time to receive the full response from the server
        ```
        upstream stream_backend {
            least_time first_byte;
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12346;
        }
        ```

    * *Hash* – NGINX selects the server based on a user‑defined key, for example, the source IP address (`$remote_addr`):

    ```
        upstream stream_backend {
            hash $remote_addr;
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12346;
        }
        ```
        The `Hash` load‑balancing method is also used to configure session persistence. As the hash function is based on client IP address, connections from a given client are always passed to the same server unless the server is down or otherwise unavailable. Specify an optional `consistent` parameter to apply the `ketama` consistent hashing method:

        `hash $remote_addr consistent;`

    * *Random* – Each connection will be passed to a randomly selected server. If the two parameter is specified, first, NGINX randomly selects two servers taking into account server weights, and then chooses one of these servers using the specified method:
        * `least_conn` – The least number of active connections
        * `least_time=connect` (NGINX Plus) – The time to connect to the upstream server ($upstream_connect_time)
        * `least_time=first_byte` (NGINX Plus) – The least average time to receive the first byte of data from the server ($upstream_first_byte_time)
        * `least_time=last_byte` (NGINX Plus) – The least average time to receive the last byte of data from the server ($upstream_session_time)
```
        upstream stream_backend {
            random two least_time=last_byte;
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12346;
            server backend4.example.com:12346;
        }
```
The *Random* load balancing method should be used for distributed environments where multiple load balancers are passing requests to the same set of backends. For environments where the load balancer has a full view of all requests, use other load balancing methods, such as round robin, least connections and least time.

4. Optionally, for each upstream server specify server‑specific parameters including `maximum number of connections`, `server weight`, and so on:
```
    upstream stream_backend {
        hash   $remote_addr consistent;
        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
        server backend3.example.com:12346 max_conns=3;
    }
    upstream dns_servers {
        least_conn;
        server 192.168.136.130:53;
        server 192.168.136.131:53;
        # ...
    }
```
An alternative approach is to proxy traffic to a single server instead of an upstream group. If you identify the server by hostname, and configure the hostname to resolve to multiple IP addresses, then NGINX load balances traffic across the IP addresses using the `Round Robin` algorithm. In this case, you must specify the server’s port number in the `proxy_pass` directive and must not specify the protocol before IP address or hostname:
```
stream {
    # ...
    server {
        listen     12345;
        proxy_pass backend.example.com:12345;
    }
}
```

----


----


----
