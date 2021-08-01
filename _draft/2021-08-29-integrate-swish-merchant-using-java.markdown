--- 
layout: "post"
title:  "How to Integrate Swish Merchant Using Java"
date:   "2021-07-12 13:30:00 -0500"
categories: swish, payment, java, merchant, integration
---

The post describes how to quickly integrate [Swish](https://www.swish.nu/){:target="_blank"}
into your Java application using [swish-java](https://github.com/osimosu/swish-java){:target="_blank"}, a library I
developed.

The original idea for Trainingpass was a site where people can discover new activities and book them on the go, i.e no
memberships and no [startavgift](https://translate.google.com/?sl=auto&tl=en&text=startavgift&op=translate&hl=en). Users
would search for an activity e.g climbing, select single or multiple passes, then book and pay with Swish. This lead me
to looking into Swish and ultimately developing a library as I want something I can re-use in other projects, and I also
didn't want Swish specific code littered in the application.

Note that Swish provides a list of technical partners you can hire you 
