This is your second step.

## Step 2 - Instrumenting one single function

In this first step, we'll use basic manual instrumentation to trace one single function from our application. 

First, we configure the agent to make it receive traces. 
```yaml
# docker-compose.yaml

  agent:
    image: "datadog/agent:latest"
    environment:
      - DD_API_KEY
      - DD_APM_ENABLED=true
      - DD_APM_NON_LOCAL_TRAFFIC=TRUE
    ports: 
      - "8126:8126"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /proc/:/host/proc/:ro
      - /sys/fs/cgroup/:/host/sys/fs/cgroup:ro
```

Then, let's instrument the code. The first thing to do is to import and configure tracing capabilities:

```python
# cafe.py

from ddtrace import tracer, config, patch_all;
tracer.configure(hostname='agent', port=8126)
```

After what, we instrument the `beers()` function by adding the tracer decorator.
```python
# cafe.py

@app.route('/beers')
@tracer.wrap(service='beers')
def beers():
```

Now, when you call your webapp for beers `curl -XGET "localhost:5000/beers"`, you see the traces of the underlying `beers()` function in Datadog Trace List [Datadog Trace List](https://app.datadoghq.com/apm/traces) 

When you click (View Trace ->) on your newly tracked service, you see the "details" of the trace. For now, the details are limited to the single span of the `beers` method you just instrumented. One valuable information you find here is the duration of the span.

If you access the [Beer Service Statistics](https://app.datadoghq.com/apm/service/beers/app.beers) page, you also find statistics about all the occurences of calls to that service (try `curl -XGET "localhost:5000/beers"` ten times in a row to populate that statistics. 

This is useful, but you'll need more to observe what's happening in your application and eventually fix or optimize things.
