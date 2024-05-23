---
title: "How This Blog Migrated to Cloudflare Pages"
date: 2024-05-23T15:17:17+07:00
draft: false
toc: true
images:
tags:
  - cloudflare
  - github
---

Recently i've learn nginx for my freelance project this provoked me to learn a little further about nginx. After learn some topics about nginx i decide to make this blog and deploy it using AWS EC2 instance and Nginx [How This Blog Deployed](https://blog.alfarizzi.xyz/posts/how-this-blog-deployed/) and this cost me a lot 🥲. so i decide to move my blog to Cloudflare Pages (🎉🎉 Learn new thing). this how i do it.

## Create Cloudflare Account
![Register Cloudflare](../images/register-cloudflare.png)

Before start this journey you need to register new account using your email or you can sign in using apple account. You will received an email for email verication in your inbox.

## Create New Application
![Connect to Git](../images/connect-to-git.png)

Go to `Workers & Pages` > `Pages` > `Connect to Git` then you will be directed to github/gitlab to configure the repos you that will you manage using cloudflare.

![Select Orgs](../images/select-orgs.png)

then you will be redirected to page where you can create new application.

![Create New Application](../images/create-new-app.png)

select repository you want to deploy, here i'll choose `blog` repo.
![Choose Repo](../images/choose-repo.png)

since this blog is built using `hugo` then choose `hugo` as framework preset and leave other option as default.
![Setting Build Configuration](../images/build-config.png)

then click `Save and Deploy` then deployment process will be started.

![Deployment](../images/deployment.png)

if deployment process you will get random generated url that pointed to your deployed project in this you also can setting custom domain/subdomain for this.