---
title: "How This Blog Deployed"
date: 2024-05-18T14:31:08+07:00
draft: false
toc: true
images:
tags:
  - AWS EC2
  - Nginx
  - SSH
---

## AWS EC2
This blog is deployed using aws ec2 instance. AWS EC2 instance is a web service that provides sizeable compute capicity in the clouds. instance type for ths blog is `t3.micro`.

## Nginx
Under the hood, this blog using `hugo`, a website generator that produces static files (HTML, CSS, JS) for the final result. these files are served using `nginx` with the configuration below:
```conf
events {}

http {
    include mime.types;

    server {
      server_name <your domain>;
      location / {
        root <path to blog dir>/blog/public;
        index index.html index.htm;
        try_files $uri $uri/ =404;
      }
  }
}
```
## SSH
SSH (Secure Shell) is a protocol to send command to securely sending commands to a server. Imagine line store owner ask the employee to clean/close/repair the store.
## Github Action
Github Action is CI/CD tools that help you automate your workflows like testing, build, and deployment. this blog leverage github action for deployment processs

## How it Works
When i done write some posts, i need to run `hugo` command on my local machine to produce static files then push it to github repository, deployment workflow will run automatically when some changes is pushed to master branch. Deployment script will do `ssh` to my `aws ec2` instance and pull latest changes from master branch, below is full script for deployment.

```yaml
name: Deploy

on:
  push:
    branches:
      - master

jobs:
  Deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v2 
      - name: Build & Deploy
        env:
            PRIVATE_KEY: ${{secrets.SSH_PRIVATE_KEY}}
            HOSTNAME: ${{secrets.SSH_HOST}}
            USER_NAME: ${{secrets.SSH_USER_NAME}}
      
        run: |
          echo "$PRIVATE_KEY" > private_key && chmod 600 private_key
          ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOSTNAME} '
              cd blog &&
              git pull origin master &&
              sudo cp -r public <path to blog>/blog
              '
```