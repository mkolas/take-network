+++
title = "Docker"
description = ""
tags = [
    "programming",
    "python",
    "golang",
    "2017"
]
date = "2018-01-06"
categories = [
    "Programming",
]
menu = "main"
+++

docker pull jenkins/jenkins:lts

docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

available on localhost:8080