---
layout: post
title: Distributed Job Processing with Celery
categories: Distributed Job Processing with Celery
description: Distributed Job Processing with Celery. Celery is written in Python. Celery is an asynchronous task queue/job queue based on distributed message passing.
---

Recently I got a task to create a job processing application. The application will fetch the data from an API, parse the data, and create jobs for the data.

The job is to record a livestream using ffmpeg. There will be multiple jobs running simultaneously, so the idea is to create a distributed job processing application.

We have selected [Celery](http://www.celeryproject.org/) as it is an asynchronous distributed task queue written in Python. Also we have used [RabbitMQ](http://www.rabbitmq.com/) as the message broker.

We have designed the infrastructure with the application running on multiple instances. One instance will be the Celery master where we execute the script to fetch data from API, parse the data, add jobs to the queue. Other instances will be running the Celery worker with the same RabbitMQ server as message broker.

    app = Celery('tasks', backend='amqp://guest@192.168.1.3//', 
                           broker='amqp://guest@192.168.1.3//')

We are using a cron job in the Celery master to execute the script periodically. The Celery master and the Celery workers will be pointing to the same RabbitMQ server, so whenever a new job is added to the queue it will be assigned to a worker.

We are also using [Flower](https://github.com/mher/flower): a celery monitoring tool in Celery master to check the status of the jobs and for other monitoring.
