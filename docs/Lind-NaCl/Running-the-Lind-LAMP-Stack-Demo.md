## Test Description

The test sets up and runs the entire LAMP stack (Postgres, Nginx, Python w/ Gunicorn + Flask) using Lind, serving a dynamic HTML page of 2MB. Each request sent on flask generates 2048 database rows and performs INSERT, SELECT and DELETE on the rows.

## Launching the Lind LAMP stack

Currently only works on Linux

To launch the Lind LAMP Stack you can run:

```
docker pull securesystemslab/lind-lamp 
docker run --privileged --ipc=host --cap-add=SYS_PTRACE --cap-add=SYS_ADMIN --name=demo_lind_lamp -p 8888:80 -it securesystemslab/lind-lamp /bin/bash
```

The LAMP Stack is ready to connect when you see gunicorn print:

```
[2024-04-24 22:29:13 +0000] [28] [INFO] Booting worker with pid: 28
```

## Connecting to the Lind LAMP Stack

From another terminal you can curl or run the [wrk benchmarker](https://github.com/wg/wrk.git)


You can run curl to run a quick test which will generate our example 2MB html file.

```
curl http://localhost:8888/ > out.html 
```

You can run the benchmark using wrk with the following command:

```
wrk -t1 -c1 -d30 --timeout 30 http://localhost:8888/
```

This will yield benchmarking results in the form of:

```
Running 30s test @ http://localhost:8888/
  1 threads and 1 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   637.44ms  215.59ms   1.04s    58.70%
    Req/Sec     1.37      0.68     3.00     89.13%
  46 requests in 30.03s, 96.33MB read
Requests/sec:      1.53
Transfer/sec:      3.21MB

```






