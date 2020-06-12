# Notes I've learned about HTTP, Web Servers/Frameworks

## Part 1 - Basic HTTP protocol
- source: https://ruslanspivak.com/lsbaws-part1
- http://localhost:8888/hello
- this asks for the http protocol by forming a TCP connection to the web server,
    which responds and sends something to the 
- http:// is HTTP protocol
- localhost is the name of the host
- 8888 is the port number, like a room in the house
- /hello is the path of the port

## Part 2 - WSGI and Mixing Web Servers and Frameworks
- can use WSGI to let the server use all web frameworks like Flask, Django, and Pyramid
- WSGI stands for Web Server Gateway Interface
- this is interface between Python Web Servers and Python Web Frameworks
- tell the server to load the 'app' callable from the Python module
- then this loaded app shows up when the server takes a request and returns this to the HTTP
- specified the /hello which shows up at the end of the url
- when you run the server and call it by typing in the url, it generates a response
    that transmits to the client, HTTP headers that web server should generally have is 'response_headers'
    - Content-Type (text/plain, or charset = utf-8)
    - Content-Length
    - Date
    - Server
- 'environ' dictionary is Python dictionary that contains WSGI and CGI variables
- all the values in the environ dictionary are taken from parsing the HTTP request
- then the web framework uses the info in this dictionary
    1. Server starts and loads the 'app' callable (provided from the command line parameter of the web application file)
    2. Server reads a request
    3. Server parses request
    4. Builds the 'environ' dictionary using this parsed data
    5. Then, it calls the 'app' callable using, a) environ dictionary and b) start_response callable 
        and gets a response body back that is used in next step
    6. Server uses the data from the 'app' call and data from the start_response(this includes the status and 
        response headers), to construct an HTTP response
    7. Server transmits this HTTP response back to the client

## Part 3 - Sockets and listen
- a socket allows the program to communicate with another program
- the socket is the combination is IP address and the port
- socket pair for a TCP connection is a 4-tuple that identifies local IP, local port, foreign IP, foreign port
- server sequence to create socket and accept client connection:
    1. creates TCP/IP socket 
    ```listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)```
    2. can set some options for the socket
    3. server binds or assigns local protocol address 
    ```listen_socket.bind(SERVER_ADDRESS)```
    4. server makes the socket a listening socket 
    ```listen_socket.listen(REQUEST_QUEUE_SIZE)```
    this is called by server to accept incoming connection requests for this specific socket
    5. when there is a connection from the client, the accept call returns the connected client socket
- client doesn't need to call bind or accept
- local port of the client = ephemeral port (short-lived)
- specific port # that identifies as a well-known service is a well-known port

## Part 4 - Processes and File Descriptors
- kernel records information about the process that is run when you run the server py file
- file descriptor is non-negative integer that kernel returns to a process when it, identifies it
    - opens an existing file
    - creates a new file
    - creates a new socket
- these file descriptors are how you can identify a file under the hood, but in the high level you use socket objects
    to work with them
- default file descriptor: 0 (stdin), 1 (stdout), 2 (stderr)
- apparently everything in UNIX is a file
- socket also has a file descriptor 
    ```socket.listen(REQUEST_QUEUE_SIZE)``` 
    this parameter is the BACKLOG argument that determines the size of the 
    queue for incoming connection requests, have a larger BACKLOG so that accept call doesn't need to wait until new connection
    is ready, but accepts it so that it is on the queue to get served

## Part 5 - Concurrent Servers, Parent and Child Processes
- How to make a concurrent server? (handle more than one request)
- use the Unix fork()
- now there is a parent and a child process, and even though the parent process closes the client connection, the child process
    can still read the client socket data because of the descriptor reference count
- only when descriptor count = 0, the socket is closed
- ex: when there is one parent and one child, file descriptors are copied over and reference count is incremented so now
    descriptor reference for the socket = 2
    - when the parent closes client connection, this descriptor reference becomes 1
- parent process uses if(pid == 0) to see if the process is the child, and then handles the request in the child, not the parent
- two events are concurrent if you can't tell by looking at the program which will happen first, so at the exact same time
- after running and the child process closes the client connection, curl may not terminate because of these descriptor references
    that were duplicated over when you copied it

## Part 6 - Problems with the duplicate descriptors
- two problems: must close the client socket in the child process, and there can be zombies!
- if the duplicate descriptors aren't closed, then the clients won't terminate and no more file descriptors available (# open files)
- zombies can't be killed by the terminal and have the <defunct> on their process ps
- zombies are created when the parent process has not waited for the child process and that process has already terminated
- this clogs up everything and then you can't create anymore child processes
- to get around this, you have to tell the parent to wait for the child to get to its termination status
- can use the signal handler with the wait system call
    - when the child process finishes, kernel sends a SIGCHILD signal
    - parent process has a signal handler that waits for this child to get the termination status
- problem when the parent is blocked by the signal handler from accept another connection
- solve this by using try catch, make sure it is able to accept; if not you can restart accept 
- when you run a bunch of processes though, they are not queued so you get a bunch of SIGCHILD signals being sent to the parent
- this means the server can miss a few of these signals and then there are zombies again
- to fix, instead of using wait, do 
    ```waitpid(-1, WNOHANG)```
    to make sure all the terminated process signals are received
