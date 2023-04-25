# Lab Report 2

## Part 1 - String Server

For part 1 of the lab report, the assignment was to create a web server called String Server that keeps track of a single string that gets added to by incoming requests. I created this server by expanding off the web server that I created in the Week 2 Lab. 

The code for the lab consists of two files, Server.java and StringServer.java, as can be seen below: cv

### Server.java (supplied as is from the Week 2 Lab)

``` // A simple web server using Java's built-in HttpServer

// Examples from https://dzone.com/articles/simple-http-server-in-java were useful references

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

interface URLHandler {
    String handleRequest(URI url);
}

class ServerHttpHandler implements HttpHandler {
    URLHandler handler;
    ServerHttpHandler(URLHandler handler) {
      this.handler = handler;
    }
    public void handle(final HttpExchange exchange) throws IOException {
        // form return body after being handled by program
        try {
            String ret = handler.handleRequest(exchange.getRequestURI());
            // form the return string and write it on the browser
            exchange.sendResponseHeaders(200, ret.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(ret.getBytes());
            os.close();
        } catch(Exception e) {
            String response = e.toString();
            exchange.sendResponseHeaders(500, response.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(response.getBytes());
            os.close();
        }
    }
}

public class Server {
    public static void start(int port, URLHandler handler) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        //create request entrypoint
        server.createContext("/", new ServerHttpHandler(handler));

        //start the server
        server.start();
        System.out.println("Server Started! Visit http://localhost:" + port + " to visit.");
    }
}
```

### StringServer.java (
```
import java.io.IOException;
import java.net.URI;

class Handler implements URLHandler {
    String savedStrings = "";

    public String handleRequest(URI url) {
        //Home page
        if (url.getPath().equals("/")) {
            return String.format("Welcome to string server! Use /add-message?s=<string> to add to the list of strings stored on this server");
        }
        else 
        {
            System.out.println("Path: " + url.getPath());
            //Add a string to the list
            if (url.getPath().contains("/add-message")) {
                String[] parameters = url.getQuery().split("=");
                if (parameters[0].equals("s")) {
                    //First message
                    if (savedStrings == "")
                    {
                        savedStrings = parameters[1];
                    }
                    //Later messages
                    else
                    {
                        savedStrings = savedStrings + "\n" + parameters[1];
                    }

                    return savedStrings;
                }
            }
            
            //Else if invalid address return error message
            return "404 Not Found!";
        }
    }
}

class StringServer {
    public static void main(String[] args) throws IOException {
        if(args.length == 0){
            System.out.println("Missing port number! Try any number between 1024 to 49151");
            return;
        }

        int port = Integer.parseInt(args[0]);

        Server.start(port, new Handler());
    }
}
```
