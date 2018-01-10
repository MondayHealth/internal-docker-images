# Installation
Just, you know, install docker.

# Use
1. Make changes.
1. `docker build -t x509-nginx .`
1. `docker tag x509-nginx:latest 685733022723.dkr.ecr.us-east-2.amazonaws.com/x509-nginx-tools:v< NEW V NUMBER >`
1. `docker push 685733022723.dkr.ecr.us-east-2.amazonaws.com/x509-nginx-tools:v< SAME V NUMBER >`
1. Go to the ECS console, stop the current nginx task
1. Change the service and it should reload the task automagically

