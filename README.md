
# Setup amqp pub/sub clients in a docker container
* build the images in `cri` unless they are already built
* currently the rpi image is at mak3r/amqp-tools:rpi

## install rabbitmq and services
1. create the k8s resources defined in k8s-resources
### There is a consumer deployment but it is just an example.
* The same deployment can be used for consuming or producing.
* Use the content below to produce or consume from the amqp-tools image or from a pod hosting that container.


## amqp publish
### amqp publish 100 messages
`for i in {0..100}; do amqp-publish -u amqp://guest:guest@4bplus001 -e demo.logs -r "key" -b "Hello $i \n"; done`

### amqp publish 100 messages from a script
`talk.sh`:
```
echo Hello $1
```
`for i in {0..100}; do talk=$(./talk.sh "$i"); amqp-publish -u amqp://guest:guest@4bplus001 -e demo.logs -r "key" -b "$talk"; done`

## amqp consume
### continuously consume
`amqp-consume -u amqp://guest:guest@4bplus001 -e demo.logs -r "key" cat`
### consume 25 messages
`amqp-consume -u amqp://guest:guest@4bplus001 -e demo.logs -r "key" -c 25 cat`
### consume continuously and specify a queue name
`amqp-consume -u amqp://guest:guest@4bplus001 -e demo.logs -r "key" -q "k8s-consumer" cat`

## links
https://www.systutorials.com/docs/linux/man/1-amqp-consume/
https://www.systutorials.com/docs/linux/man/1-amqp-publish/
https://hub.docker.com/_/rabbitmq
https://www.rabbitmq.com/management.html

