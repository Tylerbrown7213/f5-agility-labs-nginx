Hello World [http/hello]
========================

As is customary for any programming class, our first lab outputs "Hello World!"  When acting as a web server, NGINX serves static content from files stored on disk.  By using the `js_content` directive NGINX can serve dynamic content generated by JavaScript code.  In this example, our code simply returns the string "Hello World!" and sets the return code to 200.

**Step 1:** Copy and paste the following commands to start your NGINX container with this lab's files:


.. code-block:: shell

  EXAMPLE='http/hello'
  docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro -v $(pwd)/njs/:/etc/nginx/njs/:ro -p 80:80 -d nginx

**Step 2:** Now let's use curl to test our NGINX server:

.. code-block:: none
  :emphasize-lines: 1,4,7

  curl http://localhost/
  Hello world!

  curl http://localhost/version
  0.5.3

  docker stop njs_example

Code Snippets
~~~~~~~~~~~~~

Our JavaScript code is in two files so we need two `js_import` lines to load them into our NGINX configuration.  Notice in the `js_content` directives how we use a namespace to identify in which file a JavaScript function is located.

.. code-block:: nginx
  :linenos:
  :caption: nginx.conf

  load_module modules/ngx_http_js_module.so;

  events {}

  http {
    js_path "/etc/nginx/njs/";

    js_import utils.js;
    js_import main from http/hello.js;

    server {
      listen 80;

      location = /version {
         js_content utils.version;
      }

      location / {
        js_content main.hello;
      }
    }
  }

The Javascript code to generate "Hello World!":

.. code-block:: js
  :linenos:
  :caption: hello.js

  function hello(r) {
    r.return(200, "Hello world!\n");
  }

  export default {hello}

The utils.js file will be part of all of our labs.  It has one function that displays the version of the njs module in use on this NGINX server.

.. code-block:: js
  :linenos:
  :caption: utils.js

  function version(r) {
      r.return(200, njs.version);
  }

  export default {version}

