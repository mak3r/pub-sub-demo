
# Setup amqp pub/sub clients in a docker container
* build the images in `cri` unless they are already built
* currently the rpi image is at mak3r/amqp-tools:rpi

## install rabbitmq and services in a pod inside k8s
1. Create the k8s resources defined in k8s-resources
1. Create a fanout exchange 
Note: if the pod is redeployed this needs to be re-run. To resolve this, put this in rabbitmq startup config
    * get the pod name for rabbitmq
    * `POD=$(kubectl get pod -l app=rabbitmq-ns-rabbitmq -n rabbitmq-ns -o jsonpath="{.items[0].metadata.name}")`
    * create the exchange
    * `kubectl exec -c rabbitmq -n rabbitmq-ns $POD -- /usr/local/bin/rabbitmqadmin declare exchange name=demo.logs type=fanout`
1. Build the services required to access rabbitmq
    1. 

### There are consumer and producer deployments but they are just examples.
* The same deployment can be used for consuming or producing.
* Use the content below to produce or consume from the amqp-tools image or from a pod hosting that container.


# Produce and Consume to/from pods in the cluster
## amqp publish
### amqp publish 100 messages
`for i in {0..100}; do amqp-publish -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -b "Hello $i \n"; done`

### amqp publish 100 messages from a script
`talk.sh`:
```
echo Hello $1
```
`for i in {0..100}; do talk=$(./talk.sh "$i"); amqp-publish -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -b "$talk"; done`

## amqp consume
### continuously consume
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" cat`
### consume 25 messages
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -c 25 cat`
### consume continuously and specify a queue name
`amqp-consume -u amqp://guest:guest@rabbitmq.rabbitmq-ns.svc -e demo.logs -r "key" -q "k8s-consumer" cat`

--- 

# Produce and Consume to/from the hostname (for external clients)
*You must expose the HostPort as a Kubernetes Service for this method*

## amqp publish
### amqp publish 100 messages
`for i in {0..100}; do amqp-publish -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -b "Hello $i \n"; done`

### amqp publish 100 messages from a script
`talk.sh`:
```
echo Hello $1
```
`for i in {0..100}; do talk=$(./talk.sh "$i"); amqp-publish -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -b "$talk"; done`

## amqp consume
### continuously consume
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" cat`
### consume 25 messages
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -c 25 cat`
### consume continuously and specify a queue name
`amqp-consume -u amqp://guest:guest@<hostname> -e demo.logs -r "key" -q "k8s-consumer" cat`


## links
https://www.systutorials.com/docs/linux/man/1-amqp-consume/
https://www.systutorials.com/docs/linux/man/1-amqp-publish/
https://hub.docker.com/_/rabbitmq
https://www.rabbitmq.com/management.html

