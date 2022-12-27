---
layout: "post"
title:  "How to manage multiple versions of a web application using Git"
date:   "2022-12-27 15:39:00 -0500"
categories: git, template, downstream, upstream
---

Different versions of Trainingpass are deployed to different subdomains for supported countries e.g https://se.trainingpass.com for Sweden, https://dk.trainingpass.com for Denmark etc. Visiting the default site i.e https://trainingpass.com would redirect you to the right country website (based on your IP address).

In other to be able to share common code among the different versions of site, I've created a Git project that becomes a template whereby new projects can be generated from.  Most development would also happen on template repo and changes will be pulled into the generated projects. This approach also allows for custom to be implemented that specific only to the generated projects.


- Clone the template to a new project:
```
git clone git@github.com:osimosu/shared_project.git new_project
```
 - Change the remote name from origin to upstream. The `shared_project` becomes the upstream where we pull updates from:
 ```
cd new_project/
git remote -v
git remote rename origin upstream
```
- Disable ability to push code to shared_project(upstream):
```
git remote set-url --push upstream no_push
```
- Create the downstream repo in Github and add origin remote:
 ```
 git remote add origin git@github.com:osimosu/new_project.git
 git branch -M main
 git push -u origin main
```