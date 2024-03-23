<h3>Commit 1 Reflection</h3>
The handle_connection method takes in the parameter which is the mutable reference to TcpStream and then wraps it in a BufReader. Then, the next lines of the code read the lines from BufReader and collect it into the Vec<> called http_request until it hits the end of an HTTP request header, indicated by an empty line. Then, it will print the HTTP request.

<h3>Commit 2 Reflection</h3>
The handle_connection method still reads the HTTP request with BufReader and the next lines, just like in commit 1. However, this time, it also creates an HTTP response containing hello.html and sends it back to the client with that mutable TcpStream from before.

![Commit 2 screen capture](/assets/images/commit2.png)
