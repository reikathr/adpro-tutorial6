<h3>Commit 1 Reflection</h3>
The handle_connection method takes in the parameter which is the mutable reference to TcpStream. TcpStream represents the TCP connection established with the client that we can read from and write on. Then, it wraps it into a BufReader. Then, the next line reads the lines from BufReader. The lines afterward collect the retrieved lines into the Vec<> called http_request until it hits the end of an HTTP request header, indicated by an empty line (<code>.take_while(|line| !line.is_empty())</code>). Then, it will print the HTTP request for us to view on the terminal.

<h3>Commit 2 Reflection</h3>
The handle_connection method still reads the HTTP request with BufReader and the next lines, just like in commit 1. However, this time, it makes a <code>status line</code> containing "HTTP/1.1 200 OK". It also reads hello.html, turns it into a string, and puts it into <code>contents</code>. The next line stores the length of <code>contents</code>. The line afterwards combines the variables defined previously into the variable <code>response</code>. This <code>response</code> is what the program will send back to the client through that mutable TcpStream from before.

![Commit 2 screen capture](/assets/images/commit2.png)

<h3>Commit 3 Reflection</h3>
This is what the error page looks like:

![Commit 3 screen capture](/assets/images/commit3.png)

But we can do better! The refactoring step is needed because the if-else blocks that differentiate the response we create for status code 200 OK and 404 NOT FOUND essentially use the same exact logic. The only thing that differentiate the two are what triggers the block and the html file we choose to display. Hence, we can go from this:

    if request_line == "GET / HTTP/1.1" { // difference point
        let status_line = "HTTP/1.1 200 OK"; // difference point
        let contents = fs::read_to_string("hello.html").unwrap(); // difference point
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND"; // difference point
        let contents = fs::read_to_string("404.html").unwrap(); // last difference point. the rest is the same!
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }

to this:

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();

which is much better as it is less repetitive.

<h3>Commit 4 Reflection</h3>
In this commit, we simulate a slow response. This is done by adding another case where the program will take some time to load if it receives the request <code>GET / sleep</code>, and then opening <code>127.0.0.1:7878/</code> and <code>127.0.0.1:7878/sleep</code> in two separate windows. If we open the <code>/sleep</code> one first and try to access the <code>/</code> one, it will take time for the <code>/</code> to load because it's waiting for the <code>/sleep</code> one to load. The slow response happens because due to the program being single-threaded, it can only process one request at a time. This causes the requests to enter a queue. In this queue, the program will have to wait for a request to be finished being processed before it can get its turn. This simulation shows how bad being single-threaded can be for the program if multiple users try to access it at the same time. This behavior would be unacceptable for a real product.

<h3>Commit 5 Reflection</h3>
Threadpool is what we can use to implement multithreading. The Threadpool is a collection of threads that will run when requests come in. The Threadpool has things called Workers that will receive jobs through channels. Then, the Workers will do the jobs (process HTTP requests). While a worker is doing its job, another worker will be available to receive and do a different job. If we repeat the simulation we did in the previous commit, the program will behave normally for both windows because there will be two separate workers working on <code>/sleep</code> and <code>/</code>.
