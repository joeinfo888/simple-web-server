Simple Web Server
=================

A simple web-server, to play with the Rust programming language
The goal is not performance, but rather discovering the language and building a simple distributed system. There exists a lot of work on how to improve the performance of servers (e.g., [the C10k problem](http://en.wikipedia.org/wiki/C10k_problem)).  
The C server serves as a reference implementation: the Rust server must have the same functionalities.


Rust implementation
-------------------

The Rust implementation is located in the *rust/* directory. 
It has been successfully compiled and used with version 0.6 of the language:

    $ rustc --version
    rustc 0.6
    host: x86_64-unknown-linux-gnu

The source code is organized in the following files:

* parse_arguments.rs: a simple test on how to get and parse the command line arguments.
* read_file.rs: a simple test on how to read a file.
* echo_server.rs: a simple echo server in TCP.
* ws.rs: the simple web server

The server accepts several simultaneous connections: it spawns a new task for each accept. The *pool_size* argument has no effect.
Moreover, the server does not make the difference between "file not found" and "not authorized": it returns a 404 error in both cases.


C implementation
----------------

The C implementation is located in the *c/* directory. 
The source code is organized in the following files:  

* main.c: entry point. The main part of this file is the function *accept_connections()*, which creates the server socket.
* pool.[c+h]: the pool of threads, which accept new connections and handle the requests. The threads use the file_manager in order to get the content of the requested files.
* file_manager.[c+h]: several functions to open and manage the server files.
* util.h: some constants and debugging stuff.

One can define USE_SENDFILE in util.h in order to use the *sendfile()* system call instead of *read()* the file and then *write()* its content to the socket.


How to compile and use the server
---------------------------------

Both the C and Rust servers are compiled using the *make* command. The Makefile creates the *ws* executable in the source code folder.

To run the server:

    $ ./ws -p port -s pool_size -d web_dir

You can retrieve files using telnet:

    $ telnet server_ip server_port
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    GET /test.txt HTTP/1.0
    HTTP/1.0 200 OK
    ... 

Or using curl:

    $ curl -0 http://server_ip:server_port/test.txt

You can also use your favorite web browser, but make sure to send HTTP/1.0 requests, otherwise you will get a "505 HTTP version not supported" error.
For instance, in Firefox, enter *about:config* in the address box, search for the entry *network.http.version*, modify its value to *1.0*, and restart Firefox.


The *www* directory contains several example files to test the web server.


Evaluation
----------

We have evaluated the two implementation on a cluster of 17 machines. Details are presented [here](evaluation/README.md).
The scripts used to run the experiments can be found in the *scripts* directory.
