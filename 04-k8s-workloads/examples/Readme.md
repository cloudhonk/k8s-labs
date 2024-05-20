We will setup a basic rabbit mq consumer job, from the official [docs](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/). 

### Deploy the service

```shell
kubectl create -f https://kubernetes.io/examples/application/job/rabbitmq/rabbitmq-service.yaml
```

### Deploy the rabbitmq stateful service

```shell
kubectl create -f https://kubernetes.io/examples/application/job/rabbitmq/rabbitmq-statefulset.yaml
```

### Hydrate the rabbitmq queue

Create interactive ubuntu container

```shell
kubectl run -i --tty temp --image ubuntu:22.04

## Install rabbitmq utility commands
apt-get update && apt-get install -y curl ca-certificates amqp-tools python3 dnsutils
```

Check the network connectivity

```shell
nslookup rabbitmq-service
env | grep RABBITMQ_SERVICE | grep HOST
```

Verify by creating sample queue and consuming it.

```shell
export BROKER_URL=amqp://guest:guest@rabbitmq-service:5672
/usr/bin/amqp-declare-queue --url=$BROKER_URL -q foo -d

# publish the message
/usr/bin/amqp-publish --url=$BROKER_URL -r foo -p -b Hello

# And get it back.
/usr/bin/amqp-consume --url=$BROKER_URL -q foo -c 1 cat && echo 1>&2
```

Fill the queue with tasks

```shell
/usr/bin/amqp-declare-queue --url=$BROKER_URL -q job1  -d
for f in apple banana cherry date fig grape lemon melon
do
  /usr/bin/amqp-publish --url=$BROKER_URL -r job1 -p -b $f
done
```


### Build and load the custom docker image

```shell
docker build -t job-wq-1 .
kind load docker-image job-wq-1:latest
```

### Running the Job

```shell
kubectl apply -f ./job.yml
# The check for condition name is case insensitive
kubectl wait --for=condition=complete --timeout=300s job/job-wq-1
kubectl describe jobs/job-wq-1
```

