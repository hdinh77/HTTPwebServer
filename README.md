Notes and things I've learned about HTTP, Web Servers/Frameworks
- information from https://ruslanspivak.com/lsbaws-part1
- http://localhost:8888/hello
- this asks for the http protocol by forming a TCP connection to the web server,
    which responds and sends something to the 
- http:// is HTTP protocol
- localhost is the name of the host
- 8888 is the port number, like a room in the house
- /hello is the path of the port
- can use WSGI to let the server use all web frameworks like Flask, Django, and Pyramid
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
