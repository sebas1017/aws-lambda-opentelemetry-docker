
FIRST RUN COLLECTOR JAEGER LOCALHOST

docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -e COLLECTOR_OTLP_ENABLED=true \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 5778:5778 \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.38
--------------------------------------------------
SECOND RUN BUILD IMAGE OF LAMBDA

docker build -t lambda_function_telemetry .
--------------------------------------------------
THIRD RUN DOCKER AWS LAMBDA POINTING TO COLLECTOR OF JAEGER AND SEND TELEMETRY OF EXECUTION

docker run -e AWS_LAMBDA_EXEC_WRAPPER='/opt/python/otel-instrument' \
    -e OTEL_EXPORTER_OTLP_ENDPOINT='localhost:4317' \
    -e OTEL_EXPORTER_OTLP_INSECURE='true' \
    -e OTEL_EXPORTER_OTLP_PROTOCOL='grpc' \
    -e OTEL_EXPORTER_OTLP_TRACES_ENDPOINT='localhost:4317' \
    -e OTEL_RESOURCE_ATTRIBUTES='service.name=lambda.telemetry.service,service.version=0.1,deployment.environment=dev' \
    -e OTEL_SERVICE_NAME='lambda-telemetry-service' \
    -e OTEL_TRACING='1' \
    --network="host" \
    lambda_function_telemetry
--------------------------------------------------
TEST LAMBDA LOCALLY
curl --location 'http://localhost:8080/2015-03-31/functions/function/invocations' \
--header 'Content-Type: application/json' \
--data-raw '{"body": {}}'


--------------------------------------------------
VIEW TELEMETRY FRONTEND JAEGER UI

http://localhost:16686/search


