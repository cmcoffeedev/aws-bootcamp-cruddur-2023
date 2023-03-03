# Week 2 — Distributed Tracing

# Honeycomb
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


# Instrument AWS X-Ray for Flask


https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python.html

https://github.com/aws/aws-xray-sdk-python


First install the AWS SDK by adding the following to `backend-flask/requirements.txt`
```txt
aws-xray-sdk
```
Run the following command to install the AWS X-RAY SDK
```sh
pip install -r requirements.txt
```

Add the environment variables for the shell and gitpod
```sh
export AWS_REGION="us-west-1"
gp env AWS_REGION="us-west-1"
```

Add these imports to `backend-flask/app.py`
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```

Under the last import in `backend-flask/app.py`, add this:
```python
xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

We will set the AWS_XRAY_URL env soon in `docker-compose.yml`

## Set up X-Ray resources
Add a new JSON file `aws/json/xray.json`

```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

Set an environment variable for the flask address
```sh
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
```

Using the AWS-CLI, create a group in AWS X-RAY by running this terminal command
```sh
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\")
```

Using the AWS-CLI, create a sampling rule for AWS X-RAY using this command
```sh
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

You can also do this using the AWS GUI in AWS Cloudwatch

https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#xray:settings/groups

Notice the link above has a region as a query parameter. You may need to change this to your region. 




## Install X-Ray Daemon

https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html

https://github.com/aws/aws-xray-daemon

https://github.com/marjamis/xray/blob/master/docker-compose.yml


Add this service to your `docker-compose.yml` file above the `backend-flask` service

```yaml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-west-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp

```

Add the enviroment variables in the `backend-flask` service in `docker-compose.yml` under `BACKEND-URL`

```yaml
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

At the bottom of `.gitpod.yml` file, define the ports for each service

```yaml
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```


We can check service data for the last 10 minutes using the following command
```sh
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```

