FROM ubuntu:xenial

COPY . /app
WORKDIR /app

RUN apt-get update 
RUN apt-get install -y less \
  amqp-tools \
  vim

CMD ["/bin/bash"]