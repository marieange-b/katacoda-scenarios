This is your first step.

# Datadog Distributed Tracing Workshop
Content for a workshop on Distributed Tracing sponsored by Datadog

## Prerequisites
* First install `docker-compose`:

  `curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`` `{{execute}}

  Then modify the rights for `docker-compose`:
  `chmod +x /usr/local/bin/docker-compose`{{execute}}

* Now get the workshop material ready:
  `git clone https://github.com/DataDog/dd-py-tracing-workshop.git`{{execute}}

  And move to this directory:
  `cd dd-py-tracing-workshop`{{execute}}

* If you don't have one, create a [Datadog Account](https://app.datadoghq.com/signup) and get an ``API_KEY`` for that account (you can create from the [Datadog API page](https://app.datadoghq.com/account/settings#api)). Remember to not share this key with anyone.

## Flask Application

Here's an app that does a simple thing. It tells you what donut to pair with your craft brew. While it is contrived in its purpose,
it probably has something in common with the apps you work on:
* It's a web application that exposes HTTP endpoints.
* To do its job, it must talk to datastores and external services.
* It may need performance improvements.

## Get Started

The application runs in many Docker containers that you can launch using the following command:

`sudo DD_API_KEY=<add_your_API_KEY_here> docker-compose up -V`{{copy}}

Each Python application runs a Flask server with live-reload so you can update your code without restarting any container.
After executing the command above, you should have running:
* A Flask app ``cafe``, accepting HTTP requests
* A micro-service implemented by a smaller Flask app ``taster``, also accepting HTTP requests
* Redis, the backing datastore
* Datadog agent, a process that listens for, samples and aggregates traces

You can run the following command to verify these are running properly. You might have to use `sudo` according to your permissions.

`docker-compose ps`{{execute}}

If all containers are running properly, you should see the following:

```
            Name                           Command               State                          Ports
-----------------------------------------------------------------------------------------------------------------------------
ddpytracingworkshop_agent_1     /entrypoint.sh supervisord ...   Up      7777/tcp, 8125/udp, 0.0.0.0:8126->8126/tcp, 9001/tcp
ddpytracingworkshop_redis_1     docker-entrypoint.sh redis ...   Up      6379/tcp
ddpytracingworkshop_taster_1    python taster.py                 Up      0.0.0.0:5001->5001/tcp
ddpytracingworkshop_cafe_1      python cafe.py                   Up      0.0.0.0:5000->5000/tcp
```

Now, let's poke through the app and see how it works.

### Architecture

* Vital Business Info about Beers and Donuts lives in a SQL database.

* Some information about Donuts changes rapidly, with the waves of baker opinion, so
we store this time-sensitive information in a Redis-backed datastore called DonutStats.

* The `DonutStats` class abstracts away some of the gory details and provides a simple API

### HTTP Interface

* We can list the beers we have available
`curl -XGET "localhost:5000/beers"`{{execute}}

* The donuts we have available
`curl -XGET "localhost:5000/donuts"`{{execute}}

* We can grab a beer by name
`curl -XGET "localhost:5000/beers/ipa"`{{execute}}

* We can grab a donut by name
`curl -XGET "localhost:5000/donuts/jelly"`{{execute}}

So far so good.

Things feel pretty speedy. But what happens when we try to find a donut that pairs well with our favorite beer?

* `curl -XGET "localhost:5000/pair/beer?name=ipa"`{{execute}}

It feels slow! Slow enough that people might complain about it. Let's try to understand why.

