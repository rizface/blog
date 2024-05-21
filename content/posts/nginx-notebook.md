---
title: "Nginx Notebook"
date: 2024-05-21T20:38:13+07:00
draft: false
toc: true
images:
tags:
  - nginx
---

# Basic Config
```conf
events {}

http {
  server {
    listen 80;
    root /var/www/html;
    index index.html index.php index.htm;

    location / {
      try_files $uri $uri/ $uri?args
    }
  }
}
```
# Context

## Events
Events context is used to determine how nginx will process incoming connections, for example
- how many connection will be proceed in one time ?
- how many worker will be used to process incoming request ?
- whether all incoming connections will be processed at one time ?
and more. please read documentation below and search directives for `event` context.
[Event Context](https://nginx.org/en/docs/ngx_core_module.html#events)

```conf
worker_processes 4;
events {
  worker_connections <number>;
  use <method>; // select, poll, kqueue, rtsig, /dev/poll
  multi_accept <on | off>;
  accept_mutex <on | off>;
}
```
example above is very basic example how events context is used, below is explanation foreach directives:
- worker_connections determine how many connections each worker can accept. you need to pay attention to value of worker_processes directive, for example if worker_processes is 4 and worker_connections is 100 it mean nginx can accept 400 request simultaneously.
- use determine the event processing model (method) to be used
- multi_accept determine whether the workers will process the connections simultaneously or one by one.
- accept_mutex determine whether the workers is allowed to process same connections, if the value is on it will prevent worker process same connection and when the value is of it will allow workers to process same connections that can lead to resource wasting.