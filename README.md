
# Setup amqp pub/sub clients in a docker container
* build the images in `cri/` unless they are already built
    * `docker build -t amqp-tools:x86 .`
    * `docker build -t amqp-tools:rpi .`
* If you build your own images, of course you will also need to update the image reference in the k8s resource specs
* currently the rpi image is at mak3r/amqp-tools:rpi

## Install rabbitmq and services in a pod inside k8s
1. Create the rabbitmq namespace
    * `kubectl create ns rabbitmq-ns`
1. Create rabbit mq container
    * `kubectl apply -f k8s-resources/rabbitmq.yaml`
1. Setup a fanout exchange 
Note: if the pod is redeployed this needs to be re-run. To resolve this, put the configuration for a fanout exchange in rabbitmq startup config file (out of scope)
    * Get the pod name for rabbitmq
    * `POD=$(kubectl get pod -l app=rabbitmq-ns-rabbitmq -n rabbitmq-ns -o jsonpath="{.items[0].metadata.name}")`
    * Create the exchange
    * `kubectl exec -c rabbitmq -n rabbitmq-ns $POD -- /usr/local/bin/rabbitmqadmin declare exchange name=demo.logs type=fanout`
1. Create the clients and service required to access rabbitmq
    * `kubectl apply -f k8s-resources/rabbitmq-svc.yaml`
    * `kubectl apply -f k8s-resources/producer.yaml`
    * `kubectl apply -f k8s-resources/consumer.yaml`

### There are consumer and producer deployments but they are just examples.
* The same deployment can be used for consuming or producing.
* Use the content below to produce or consume from the amqp-tools image or from a pod hosting that container.


# Produce and Consume to/from pods in the cluster
## amqp publish
### amqp publish 100 messages
`for i in {0..100}; do amqp-publish -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -b "Hello $i"; done`

### amqp publish 100 messages from a script
`talk.sh`:
```
echo Hello $1
```
`for i in {0..100}; do talk=$(./talk.sh "$i"); amqp-publish -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -b "$talk"; done`

## amqp consume
### continuously consume
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" awk 1`
### consume 25 messages
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -c 25 awk 1`
### consume continuously and specify a queue name
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -q "k8s-consumer" awk 1`

--- 

# Produce and Consume to/from the hostname (for external clients)
*You must expose the HostPort as a Kubernetes Service for this method*

## amqp publish
### amqp publish 100 messages
`for i in {0..100}; do amqp-publish -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -b "Hello $i"; done`

### amqp publish 100 messages from a script
`talk.sh`:
```
echo Hello $1
```
`for i in {0..100}; do talk=$(./talk.sh "$i"); amqp-publish -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -b "$talk"; done`

## amqp consume
### continuously consume
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" awk 1`
### consume 25 messages
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -c 25 awk 1`
### consume continuously and specify a queue name
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -q "k8s-consumer" awk 1`


## links
https://www.systutorials.com/docs/linux/man/1-amqp-consume/
https://www.systutorials.com/docs/linux/man/1-amqp-publish/
https://hub.docker.com/_/rabbitmq
https://www.rabbitmq.com/management.html

```
annotations:
    ingress.kubernetes.io/rewrite-target: '/'
```