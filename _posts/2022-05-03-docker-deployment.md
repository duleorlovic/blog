---
layout: post
---

## AWS ECS

https://aws.amazon.com/blogs/containers/deploy-applications-on-amazon-ecs-using-docker-compose/
```
docker context create ecs myecscontext
docker context use myecscontext
docker context ls

# start containers
docker compose up
docker compose -f docker-compose.ecs.yml up

# view exposed port and url like
# http://docke-loadb-rjft57gmmqri-853572988.us-east-1.elb.amazonaws.com
docker compose ps

# remove infrastructure
docker compose down

# remove containers and clear them to free up space
docker system prune -a --volumes
```

Build and push to ECR
https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 219232999684.dkr.ecr.us-east-1.amazonaws.com
docker push 219232999684.dkr.ecr.us-east-1.amazonaws.com/selenium-ssh:strong-password 
```

