## Configuring NGINX and NGINX Plus as a Web Server
https://docs.nginx.com/nginx/admin-guide/web-server/web-server/

### Setting Up Virtual Servers

The NGINX Plus configuration file must include at least one `server` directive to define a virtual server. When NGINX Plus processes a request, it first selects the virtual server that will serve the request.

A virtual server is defined by a `server` directive in the `http` context, for example:
```
http {
    server {
        # Server configuration
    }
}
```
It is possible to add multiple `server` directives into the `http` context to define multiple virtual servers.

The `server` configuration block usually includes a `listen` directive to specify the IP address and port (or Unix domain socket and path) on which the server listens for requests. Both IPv4 and IPv6 addresses are accepted; enclose IPv6 addresses in square brackets.

The example below shows configuration of a server that listens on IP address 127.0.0.1 and port 8080:
```
server {
    listen 127.0.0.1:8080;
    # Additional server configuration
}
```
If a port is omitted, the standard port is used. Likewise, if an address is omitted, the server listens on all addresses. If the listen directive is not included at all, the “standard” port is `80/tcp` and the “default” port is `8000/tcp`, depending on superuser privileges.

If there are several servers that match the IP address and port of the request, NGINX Plus tests the request’s `Host` header field against the `server_name` directives in the `server` blocks. The parameter to server_name can be a full (exact) name, a wildcard, or a regular expression. A wildcard is a character string that includes the asterisk `*` at its beginning, end, or both; the asterisk matches any sequence of characters. NGINX Plus uses the Perl syntax for regular expressions; precede them with the tilde `~`. This example illustrates an exact name.
```
server {
    listen      80;
    server_name example.org www.example.org;
    #...
}
```
If several names match the `Host` header, NGINX Plus selects one by searching for names in the following order and using the first match it finds:

    1. Exact name
    2. Longest wildcard starting with an asterisk, such as `*.example.org`
    3. Longest wildcard ending with an asterisk, such as mail.*
    4. First matching regular expression (in order of appearance in the configuration file)

If the `Host` header field does not match a server name, NGINX Plus routes the request to the default server for the port on which the request arrived. The default server is the first one listed in the `nginx.conf` file, unless you include the `default_server` parameter to the `listen` directive to explicitly designate a server as the default.
```
server {
    listen 80 default_server;
    #...
}
```

----
### Configuring Locations

NGINX Plus can send traffic to different proxies or serve different files based on the request URIs. These blocks are defined using the `location` directive placed within a `server` directive.

For example, you can define three `location` blocks to instruct the virtual server to send some requests to one proxied server, send other requests to a different proxied server, and serve the rest of the requests by delivering files from the local file system.

NGINX Plus tests request URIs against the parameters of all location directives and applies the directives defined in the matching location. Inside each `location` block, it is usually possible (with a few exceptions) to place even more location directives to further refine the processing for specific groups of requests.

**Note:** In this guide, the word location refers to a single `location` context.

There are two types of parameter to the `location` directive: prefix strings (pathnames) and regular expressions. For a request URI to match a prefix string, it must start with the prefix string.

The following sample location with a pathname parameter matches request URIs that begin with `/some/path/`, such as `/some/path/document.html`. (It does not match `/my-site/some/path` because `/some/path` does not occur at the start of that URI.)
```
location /some/path/ {
    #...
}
```
A regular expression is preceded with the tilde `~` for case-sensitive matching, or the tilde-asterisk `~*)` for case-insensitive matching. The following example matches URIs that include the string `.html` or `.htm` in any position.
```
location ~ \.html? {
    #...
}
```

----
### NGINX Location Priority

To find the location that best matches a URI, NGINX Plus first compares the URI to the locations with a prefix string. It then searches the locations with a regular expression.

Higher priority is given to regular expressions, unless the `^~` modifier is used. Among the prefix strings NGINX Plus selects the most specific one (that is, the longest and most complete string). The exact logic for selecting a location to process a request is given below:

    1. Test the URI against all prefix strings.
    2. The `=` (equals sign) modifier defines an exact match of the URI and a prefix string. If the exact match is found, the search stops.
    3. If the `^~` (caret-tilde) modifier prepends the longest matching prefix string, the regular expressions are not checked.
    4. Store the longest matching prefix string.
    5. Test the URI against regular expressions.
    6. Stop processing when the first matching regular expression is found and use the corresponding location.
    7. If no regular expression matches, use the location corresponding to the stored prefix string.

A typical use case for the `=` modifier is requests for **/** (forward slash). If requests for / are frequent, specifying `= /` as the parameter to the location directive speeds up processing, because the search for matches stops after the first comparison.
```
location = / {
    #...
}
```
A `location` context can contain directives that define how to resolve a request – either serve a static file or pass the request to a proxied server. In the following example, requests that match the first location context are served files from the /data directory and the requests that match the second are passed to the proxied server that hosts content for the `www.example.com` domain.
```
server {
    location /images/ {
        root /data;
    }

    location / {
        proxy_pass http://www.example.com;
    }
}
```
The `root` directive specifies the file system path in which to search for the static files to serve. The request URI associated with the location is appended to the path to obtain the full name of the static file to serve. In the example above, in response to a request for `/images/example.png`, NGINX Plus delivers the file `/data/images/example.png`.

The `proxy_pass` directive passes the request to the proxied server accessed with the configured URL. The response from the proxied server is then passed back to the client. In the example above, all requests with URIs that do not start with `/images/` are be passed to the proxied server.

---
### Using Variables

You can use variables in the configuration file to have NGINX Plus process requests differently depending on defined circumstances. Variables are named values that are calculated at runtime and are used as parameters to directives. A variable is denoted by the \(\) (dollar) sign at the beginning of its name. Variables define information based upon NGINX’s state, such as the properties of the request being currently processed.

There are a number of predefined variables, such as the core HTTP variables, and you can define custom variables using the `set`, `map`, and `geo` directives. Most variables are computed at runtime and contain information related to a specific request. For example, `$remote_addr` contains the client IP address and `$uri` holds the current URI value.

----
### Returning Specific Status Codes

Some website URIs require immediate return of a response with a specific error or redirect code, for example when a page has been moved temporarily or permanently. The easiest way to do this is to use the `return` directive. For example:
```
location /wrong/url {
    return 404;
}
```
The first parameter of `return` is a response code. The optional second parameter can be the URL of a redirect (for codes 301, 302, 303, and 307) or the text to return in the response body. For example:
```
location /permanently/moved/url {
    return 301 http://www.example.com/moved/here;
}
```
The `return` directive can be included in both the `location` and `server` contexts.

----
### Rewriting URIs in Requests

A request URI can be modified multiple times during request processing through the use of the `rewrite` directive, which has one optional and two required parameters. The first (required) parameter is the regular expression that the request URI must match. The second parameter is the URI to substitute for the matching URI. The optional third parameter is a flag that can halt processing of further rewrite directives or send a redirect (code 301 or 302). For example:
```
location /users/ {
    rewrite ^/users/(.*)$ /show?user=$1 break;
}
```
As this example shows, the second parameter `users` captures though matching of regular expressions.

You can include multiple `rewrite` directives in both the `server` and `location` contexts. NGINX Plus executes the directives one-by-one in the order they occur. The `rewrite` directives in a `server` context are executed once when that context is selected.

After NGINX processes a set of rewriting instructions, it selects a `location` context according to the new URI. If the selected location contains `rewrite` directives, they are executed in turn. If the URI matches any of those, a search for the new location starts after all defined `rewrite` directives are processed.

The following example shows `rewrite` directives in combination with a return directive.
```
server {
    #...
    rewrite ^(/download/.*)/media/(\w+)\.?.*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(\w+)\.?.*$ $1/mp3/$2.ra  last;
    return  403;
    #...
}
```
This example configuration distinguishes between two sets of URIs. URIs such as /download/some/media/file are changed to /download/some/mp3/file.mp3. Because of the last flag, the subsequent directives (the second rewrite and the return directive) are skipped but NGINX Plus continues processing the request, which now has a different URI. Similarly, URIs such as /download/some/audio/file are replaced with /download/some/mp3/file.ra. If a URI doesn’t match either rewrite directive, NGINX Plus returns the 403 error code to the client.

There are two parameters that interrupt processing of rewrite directives:

* `last` – Stops execution of the `rewrite` directives in the current `server` or `location` context, but NGINX Plus searches for locations that match the rewritten URI, and any `rewrite` directives in the new location are applied (meaning the URI can be changed again).
* `break` – Like the `break` directive, stops processing of `rewrite` directives in the current context and cancels the search for locations that match the new URI. The `rewrite` directives in the new location are not executed.

----
### Rewriting HTTP Responses

Sometimes you need to rewrite or change the content in an HTTP response, substituting one string for another. You can use the `sub_filter` directive to define the rewrite to apply. The directive supports variables and chains of substitutions, making more complex changes possible.

For example, you can change absolute links that refer to a server other than the proxy:
```
location / {
    sub_filter      /blog/ /blog-staging/;
    sub_filter_once off;
}
```
Another example changes the scheme from `http://` to `https://` and replaces the `localhost` address with the hostname from the request header field. The `sub_filter_once` directive tells NGINX to apply `sub_filter` directives consecutively within a location:
```
location / {
    sub_filter     'href="http://127.0.0.1:8080/'    'href="https://$host/';
    sub_filter     'img src="http://127.0.0.1:8080/' 'img src="https://$host/';
    sub_filter_once on;
}
```
Note that the part of the response already modified with the `sub_filter` is not replaced again if another `sub_filter` match occurs.

----
### Handling Errors

With the `error_page` directive, you can configure NGINX Plus to return a custom page along with an error code, substitute a different error code in the response, or redirect the browser to a different URI. In the following example, the `error_page` directive specifies the page (`/404.html`) to return with the 404 error code.

`error_page 404 /404.html;`

Note that this directive does not mean that the error is returned immediately (the `return` directive does that), but simply specifies how to treat errors when they occur. The error code can come from a proxied server or occur during processing by NGINX Plus (for example, the `404` results when NGINX Plus can’t find the file requested by the client).

In the following example, when NGINX Plus cannot find a page, it substitutes code `301` for code `404`, and redirects the client to `http:/example.com/new/path.html`. This configuration is useful when clients are still trying to access a page at its old URI. The `301` code informs the browser that the page has moved permanently, and it needs to replace the old address with the new one automatically upon return.
```
location /old/path.html {
    error_page 404 =301 http:/example.com/new/path.html;
}
```
The following configuration is an example of passing a request to the back end when a file is not found. Because there is no status code specified after the equals sign in the `error_page` directive, the response to the client has the status code returned by the proxied server (not necessarily `404`).
```
server {
    ...
    location /images/ {
        # Set the root directory to search for the file
        root /data/www;

        # Disable logging of errors related to file existence
        open_file_cache_errors off;

        # Make an internal redirect if the file is not found
        error_page 404 = /fetch$uri;
    }

    location /fetch/ {
        proxy_pass http://backend/;
    }
}
```
The `error_page` directive instructs NGINX Plus to make an internal redirect when a file is not found. The `$uri` variable in the final parameter to the `error_page` directive holds the URI of the current request, which gets passed in the redirect.

For example, if `/images/some/file` is not found, it is replaced with `/fetch/images/some/file` and a new search for a location starts. As a result, the request ends up in the second location context and is proxied to http://backend/.

The `open_file_cache_errors` directive prevents writing an error message if a file is not found. This is not necessary here since missing files are correctly handled.

----
----
## Serving Static Content
### Root Directory and Index Files

The `root` directive specifies the root directory that will be used to search for a file. To obtain the path of a requested file, NGINX appends the request URI to the path specified by the `root` directive. The directive can be placed on any level within the `http {}`, `server {}`, or `location {}` contexts. In the example below, the root directive is defined for a virtual server. It applies to all `location {}` blocks where the `root` directive is not included to explicitly redefine the root:
```
server {
    root /www/data;

    location / {
    }

    location /images/ {
    }

    location ~ \.(mp3|mp4) {
        root /www/media;
    }
}
```
Here, NGINX searches for a URI that starts with `/images/` in the `/www/data/images/` directory in the file system. But if the URI ends with the `.mp3` or `.mp4` extension, NGINX instead searches for the file in the `/www/media/` directory because it is defined in the matching `location` block.

If a request ends with a slash, NGINX treats it as a request for a directory and tries to find an index file in the directory. The index directive defines the index file’s name (the default value is `index.html`). To continue with the example, if the request URI is `/images/some/path/`, NGINX delivers the file `/www/data/images/some/path/index.html` if it exists. If it does not, NGINX returns HTTP code `404 (Not Found)` by default. To configure NGINX to return an automatically generated directory listing instead, include the on parameter to the `autoindex` directive:
```
location /images/ {
    autoindex on;
}
```
You can list more than one filename in the `index` directive. NGINX searches for files in the specified order and returns the first one it finds.
```
location / {
    index index.$geo.html index.htm index.html;
}
```
The `$geo` variable used here here is a custom variable set through the `geo` directive. The value of the variable depends on the client’s IP address.

To return the index file, NGINX checks for its existence and then makes an internal redirect to the URI obtained by appending the name of the index file to the base URI. The internal redirect results in a new search of a location and can end up in another location as in the following example:
```
location / {
    root /data;
    index index.html index.php;
}

location ~ \.php {
    fastcgi_pass localhost:8000;
    #...
}
```
Here, if the URI in a request is `/path/`, and `/data/path/index.html` does not exist but `/data/path/index.php` does, the internal redirect to `/path/index.php` is mapped to the second location. As a result, the request is proxied.

----
### Trying Several Options
The `try_files` directive can be used to check whether the specified file or directory exists; NGINX makes an internal redirect if it does, or returns a specified status code if it doesn’t. For example, to check the existence of a file corresponding to the request URI, use the `try_files` directive and the `$uri` variable as follows:
```
server {
    root /www/data;

    location /images/ {
        try_files $uri /images/default.gif;
    }
}
```
The file is specified in the form of the URI, which is processed using the `root` or `alias` directives set in the context of the current location or virtual server. In this case, if the file corresponding to the original URI doesn’t exist, NGINX makes an internal redirect to the URI specified by the last parameter, returning `/www/data/images/default.gif`.

The last parameter can also be a status code (directly preceded by the equals sign) or the name of a location. In the following example, a `404` error is returned if none of the parameters to the `try_files` directive resolve to an existing file or directory.
```
location / {
    try_files $uri $uri/ $uri.html =404;
}
```
In the next example, if neither the original URI nor the URI with the appended trailing slash resolve into an existing file or directory, the request is redirected to the named location which passes it to a proxied server.
```
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://backend.example.com;
}
```

----
### Optimizing Performance for Serving Content

Loading speed is a crucial factor of serving any content. Making minor optimizations to your NGINX configuration may boost the productivity and help reach optimal performance.

----
### Enabling sendfile
By default, NGINX handles file transmission itself and copies the file into the buffer before sending it. Enabling the `sendfile` directive eliminates the step of copying the data into the buffer and enables direct copying data from one file descriptor to another. Alternatively, to prevent one fast connection from entirely occupying the worker process, you can use the `sendfile_max_chunk` directive to limit the amount of data transferred in a single `sendfile()` call (in this example, to `1` MB):
```
location /mp3 {
    sendfile           on;
    sendfile_max_chunk 1m;
    #...
}
```

----
### Enabling tcp_nopush
Use the `tcp_nopush` directive together with the `sendfile on;` directive. This enables NGINX to send HTTP response headers in one packet right after the chunk of data has been obtained by `sendfile()`.
```
location /mp3 {
    sendfile   on;
    tcp_nopush on;
    #...
}
```

----
### Enabling tcp_nodelay
The `tcp_nodelay` directive allows override of `Nagle’s algorithm`, originally designed to solve problems with small packets in slow networks. The algorithm consolidates a number of small packets into a larger one and sends the packet with a `200` ms delay. Nowadays, when serving large static files, the data can be sent immediately regardless of the packet size. The delay also affects online applications (ssh, online games, online trading, and so on). By default, the `tcp_nodelay` directive is set to on which means that the Nagle’s algorithm is disabled. Use this directive only for `keepalive` connections:
```
location /mp3  {
    tcp_nodelay       on;
    keepalive_timeout 65;
    #...
}
```

----
### Optimizing the Backlog Queue
One of the important factors is how fast NGINX can handle incoming connections. The general rule is when a connection is established, it is put into the “listen” queue of a listen socket. Under normal load, either the queue is small or there is no queue at all. But under high load, the queue can grow dramatically, resulting in uneven performance, dropped connections, and increased latency.
Displaying the Listen Queue

To display the current listen queue, run this command: `netstat -Lan`

The output might be like the following, which shows that in the listen queue on port `80` there are `10` unaccepted connections against the configured maximum of 128 queued connections. This situation is normal.
```
Current listen queue sizes (qlen/incqlen/maxqlen)
Listen         Local Address         
0/0/128        *.12345            
10/0/128        *.80       
0/0/128        *.8080
```
In contrast, in the following command the number of unaccepted connections (`192`) exceeds the limit of `128`. This is quite common when a web site experiences heavy traffic. To achieve optimal performance, you need to increase the maximum number of connections that can be queued for acceptance by NGINX in both your operating system and the NGINX configuration.
```
Current listen queue sizes (qlen/incqlen/maxqlen)
Listen         Local Address         
0/0/128        *.12345            
192/0/128        *.80       
0/0/128        *.8080
```
### Tuning the Operating System
Increase the value of the `net.core.somaxconn` kernel parameter from its default value (`128`) to a value high enough for a large burst of traffic. In this example, it’s increased to `4096`.

1. Run the command: `sudo sysctl -w net.core.somaxconn=4096`
2. Use a text editor to add the following line to `/etc/sysctl.conf`: `net.core.somaxconn = 4096`

### Tuning NGINX
If you set the `somaxconn` kernel parameter to a value greater than 512, change the backlog parameter to the NGINX listen directive to match:
```
server {
    listen 80 backlog=4096;
    # ...
}
```

----
----

## NGINX Reverse Proxy
### Passing a Request to a Proxied Server

When NGINX proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client. It is possible to proxy requests to an HTTP server (another NGINX server or any other server) or a non-HTTP server (which can run an application developed with a specific framework, such as PHP or Python) using a specified protocol. Supported protocols include `FastCGI`, `uwsgi`, `SCGI`, and `memcached`.

To pass a request to an HTTP proxied server, the `proxy_pass` directive is specified inside a `location`. For example:
```
location /some/path/ {
    proxy_pass http://www.example.com/link/;
}
```
This example configuration results in passing all requests processed in this location to the proxied server at the specified address. This address can be specified as a domain name or an IP address. The address may also include a port:
```
location ~ \.php {
    proxy_pass http://127.0.0.1:8000;
}
```
Note that in the first example above, the address of the proxied server is followed by a URI, `/link/`. If the URI is specified along with the address, it replaces the part of the request URI that matches the location parameter. For example, here the request with the `/some/path/page.html` URI will be proxied to http://www.example.com/link/page.html. If the address is specified without a URI, or it is not possible to determine the part of URI to be replaced, the full request URI is passed (possibly, modified).

To pass a request to a non-HTTP proxied server, the appropriate `**_pass` directive should be used:

* `fastcgi_pass` passes a request to a FastCGI server
* `uwsgi_pass` passes a request to a uwsgi server
* `scgi_pass` passes a request to an SCGI server
* `memcached_pass` passes a request to a memcached server

Note that in these cases, the rules for specifying addresses may be different. You may also need to pass additional parameters to the server (see the reference documentation for more detail).

The `proxy_pass` directive can also point to a `named group` of servers. In this case, requests are distributed among the servers in the group according to the specified method.

----
### Passing Request Headers

By default, NGINX redefines two header fields in proxied requests, “Host” and “Connection”, and eliminates the header fields whose values are empty strings. “Host” is set to the `$proxy_host` variable, and “Connection” is set to close.

To change these setting, as well as modify other header fields, use the `proxy_set_header` directive. This directive can be specified in a `location` or higher. It can also be specified in a particular `server` context or in the `http` block. For example:
```
location /some/path/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://localhost:8000;
}
```
In this configuration the “Host” field is set to the `$host` variable.

To prevent a header field from being passed to the proxied server, set it to an empty string as follows:
```
location /some/path/ {
    proxy_set_header Accept-Encoding "";
    proxy_pass http://localhost:8000;
}
```

----
### Configuring Buffers

By default NGINX buffers responses from proxied servers. A response is stored in the internal buffers and is not sent to the client until the whole response is received. Buffering helps to optimize performance with slow clients, which can waste proxied server time if the response is passed from NGINX to the client synchronously. However, when buffering is enabled NGINX allows the proxied server to process responses quickly, while NGINX stores the responses for as much time as the clients need to download them.

The directive that is responsible for enabling and disabling buffering is proxy_buffering. By default it is set to on and buffering is enabled.

The proxy_buffers directive controls the size and the number of buffers allocated for a request. The first part of the response from a proxied server is stored in a separate buffer, the size of which is set with the proxy_buffer_size directive. This part usually contains a comparatively small response header and can be made smaller than the buffers for the rest of the response.

In the following example, the default number of buffers is increased and the size of the buffer for the first portion of the response is made smaller than the default.
```
location /some/path/ {
    proxy_buffers 16 4k;
    proxy_buffer_size 2k;
    proxy_pass http://localhost:8000;
}
```
If buffering is disabled, the response is sent to the client synchronously while it is receiving it from the proxied server. This behavior may be desirable for fast interactive clients that need to start receiving the response as soon as possible.

To disable buffering in a specific location, place the proxy_buffering directive in the location with the off parameter, as follows:
```
location /some/path/ {
    proxy_buffering off;
    proxy_pass http://localhost:8000;
}
```
In this case NGINX uses only the buffer configured by proxy_buffer_size to store the current part of a response.

A common use of a reverse proxy is to provide load balancing. Learn how to improve power, performance, and focus on your apps with rapid deployment in the free Five Reasons to Choose a Software Load Balancer ebook.

----
### Choosing an Outgoing IP Address
If your proxy server has several network interfaces, sometimes you might need to choose a particular source IP address for connecting to a proxied server or an upstream. This may be useful if a proxied server behind NGINX is configured to accept connections from particular IP networks or IP address ranges.

Specify the `proxy_bind` directive and the IP address of the necessary network interface:
```
location /app1/ {
    proxy_bind 127.0.0.1;
    proxy_pass http://example.com/app1/;
}

location /app2/ {
    proxy_bind 127.0.0.2;
    proxy_pass http://example.com/app2/;
}
```
The IP address can be also specified with a variable. For example, the `$server_addr` variable passes the IP address of the network interface that accepted the request:
```
location /app3/ {
    proxy_bind $server_addr;
    proxy_pass http://example.com/app3/;
}
```


----
