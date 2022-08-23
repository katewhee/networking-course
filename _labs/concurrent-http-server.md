---
title: Concurrent HTTP Server
number: 6
repo: https://github.com/byu-ecen426-classroom/concurrent_http_server.git
---

> Controlling complexity is the essence of computer programming.
> 
> [Brian Kernighan](https://en.wikipedia.org/wiki/Brian_Kernighan){:target="_blank"}

## GitHub Classroom

1. Use the GitHub Classroom link posted in the Slack channel for the lab to accept the assignment.

2. Use [this template repository]({{ page.repo }}){:target="_blank"} to start the lab.

## Overview

For this lab, you will be extending your HTTP server to handle multiple clients _at once_. The rest of your server will stay the same. There are multiple ways of supporting multiple clients. A typical approach is to spawn a new thread or process for each client that connects. That thread or process is responsible for communicating with a specific client. There is another approach that uses a system call (`poll` or `select`) to determine which sockets are ready to receive data from or write data to. Since with socket programming, most of the time your program is waiting around for the sockets to send or receive data, this allows your program to handle multiple sockets at once. Instead of creating a new thread or process for each client socket, one process keeps track of many sockets at once.

For this lab, you will implement a concurrent server in two ways: using threads and using processes. When a new request comes in, a thread/process is created and the socket is passed to that thread/process. The newly spawned thread/process is now responsible for receiving and sending data while the main thread is still accepting new clients. As mentioned in lecture, thread/process have their own set of issues, largely shared memory. To limit these issues, try to use local variables as much as possible. You should not need to use mutex/locks in this lab!

As part of writing a well behaving server, you will need to appropriately handle the threads when you are exiting (the user hits `ctrl-c`). This allows your server to finishing handling clients that have already connected before shutting down the server. To do this, you must join all spawned threads.

Assuming you did the previous lab correctly, you shouldn't have to change any of your HTTP parsing code.

Here is a demonstration of the server:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/dnDi3XXLFpE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<div class="alert alert-warning" style="width: 560px" role="alert">
  Warning: This video is for an older version of the lab. The functionality will be the same, but some of the specifics might be slightly different.
</div>

### Command-line Interface (CLI)

To control whether you are using processes or threads, we need to modify the CLI of your program. Add two new flag options, called `--thread` and `--process`. This will control which type of concurrency you are using. The complete CLI will look like

```
Usage: http_server [--help] [-v] [-d] [-p PORT] [-f FOLDER] [--thread] [--process]

Options:
  --help
  -v, --verbose
  -d, --delay
  --port PORT, -p PORT
  --folder FOLDER, -f FOLDER
  --thread
  --process
```


### Benchmarking

As part of this lab, you will be benchmarking the different approaches to concurrency. To do so, you will be using the [wrk2](https://github.com/giltene/wrk2){:target="_blank"} tool. It sends a bunch of HTTP requests to your server and measures how long it takes to respond. You will benchmark the performance of threads vs processes. Record the results in `benchmark.md` of your repository.

For consistency, record the results of your benchmark using this configuration:

```
wrk -t2 -c100 -d30s -R2000 http://127.0.0.1:8085/index.html
```

You are welcome to play around with different configurations, but for the results you report, make sure to use this configuration.

## Objectives

- Learn how to use threading and processes in Python.

- Learn how to make an HTTP server concurrent.


## Requirements

- You must be able to handle multiple concurrent clients at once using threads and processes.

- Add the `--thread` and `--process` flags to your program.

- You must gracefully shutdown your server, waiting for all client sockets to finish.

- The default port must be `8085`.

- You must benchmark the performance of your server using threads/processes and report those results in `benchmark.md`.

- All other requirements are the same as lab 5.
 

## Testing

One way to test that your server is handling multiple clients is to add an artificial delay (e.g., `sleep`) to your program, just for testing purposes. This technique will simulate a request taking a long time and show if you are handling clients concurrently. This is already built into your CLI with the `--delay` flag.

For debugging purposes, it might be useful to prefix all log messages with the socket that you are working on. The socket number should correspond to what thread is running. This can give you insight into what each thread is doing.

The benchmarking tool, [wrk2](https://github.com/giltene/wrk2){:target="_blank"}, will also help you in your testing.


## Submission

To submit your code, push it to your Github repository. Tag the commit you want to be graded with a tag named `final`.


## Resources

- [Python Threads](https://docs.python.org/3/library/threading.html){:target="_blank"}

- [Python Precesses](https://docs.python.org/3/library/multiprocessing.html){:target="_blank"}
