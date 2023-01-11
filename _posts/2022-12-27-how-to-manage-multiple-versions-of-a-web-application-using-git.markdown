---
layout: "post"
title:  "How to manage multiple versions of a web application using Git"
date:   "2022-12-27 15:39:00 -0500"
categories: git, template, downstream, upstream
---

Different versions of [Trainingpass](https://trainingpass.com)(an hobby project of mine that allows you to search for nearby studios/activities) are deployed to different subdomains for supported countries e.g [https://se.trainingpass.com](https://se.trainingpass.com) for Sweden, [https://dk.trainingpass.com](https://dk.trainingpass.com) for Denmark etc. Visiting the default site i.e [https://trainingpass.com](https://trainingpass.com) would redirect the user to their country-specific website. The country is determined from the user's IP address.

In other to be able to share common code among the different versions of site, I've created a Git project that becomes a template whereby new projects can be generated from. Most development would also happen on template repo and changes will be pulled into the generated projects. This approach also allows for custom changes (e.g background image, query different content on homepage) to be implemented that is specific only to the generated projects/country.


- Clone the template project to a new project:
```bash
git clone git@github.com:osimosu/template_project.git new_project
```

- Change the remote name from origin to `upstream`. The `template_project` becomes the upstream where we pull updates from:
```bash
cd new_project/
git remote -v
git remote rename origin upstream
```

- Disable ability to push code to template_project(upstream):
```bash
cd new_project/
git remote set-url --push upstream no_push
```

- Create the downstream repo in Github, then manually add it as origin:
 ```bash
cd new_project/
git remote add origin git@github.com:osimosu/new_project.git
git branch -M main
git push -u origin main
```

- To pull changes from the template_project into generated project, you run:
 ```bash
 cd new_project/
git pull upstream main
```

- To pull changes from the origin, you run:
 ```bash
 cd new_project/
git pull 
```
