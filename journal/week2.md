# Week 2 — Distributed Tracing

## Honeycomb
Honeycomb uses OpenTelemetry. OpenTelemetry sets standards for telemetry data processes.
Your environment sends standarized data to honeycomb. You can also send the data to alternative backends.

Sign up for honeycomb at `https://honeycomb.io`

You start off in the test environment.

<img>

The first screen should look like this. If not click the `Datasets` tab then click `Add your data`.

Grab your API key from honeycomb.
<img>

Click the python tab and it will give instructions on how we can use this with flask
<img>

The first step is to install the dependencies in `backend-flask/requirements.txt`
```txt
opentelemetry-api
opentelemetry-sdk
opentelemetry-exporter-otlp-proto-http
opentelemetry-instrumentation-flask
opentelemetry-instrumentation-requests
```

Now run the following command
```sh
pip install -r requirements.txt
```

You can also install these by putting this in the terminal
```sh
pip install opentelemetry-api \
    opentelemetry-sdk \
    opentelemetry-exporter-otlp-proto-http \
    opentelemetry-instrumentation-flask \
    opentelemetry-instrumentation-requests
```

We also need to add the following code to `backend-flask/app.py`

First add the imports under the last import

```python
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```

We can add the rest of the code under the last import

```python
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
Add the following lines under `app = Flask(__name__)`
```python
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

Now let's update our `docker-compose.yml` file to include honeycomb for our `backend-flask` service. 

Add the following code under the `BACKEND_URL`

```dockerfile
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```

Set the environment variables for our current shell. Run these commands in the terminal
```sh
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="backend-flask"
```

Set the gitpod environment variables so they reload each time we run gitpod. Run these commands in the terminal
```sh
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="backend-flask"
```

Note: The honeycomb service name should be specific for each service. For example backend, frontend, android, iOS, etc.

