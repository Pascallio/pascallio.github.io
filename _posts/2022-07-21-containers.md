---
layout: post
title: Should containers be mandatory in computer (bio)science?
author: "Pascal Maas"
categories: IT, science, publications
image: container.png 
---

<!-- Image of containers being connected -->

<!-- Introduction into containers, chaining them into modules, language independencies -->
## Introduction
Containerization is the concept of creating a shareable virtual environment for your code. This ensures that necessary dependencies are available on any computer, making it able to create reproducible results. In science, this is an important requirement, as demonstrated by the concept of peer reviewing. Any article without peer review is potentially biased and therefore should be considered biased. Reproducible results are therefore viable.  

## Issues
<!-- Stating the problem when a container is not provided (versioning, out-of-date dependencies, etc. ) -->
One of the many reasons for the need of docker containers is version-specific dependencies. Even if versions aren't specified in requirements, updates of dependencies may break code or even worse, alter results without indications. However, declaring dependency versions is a topic for another time. A side effect of using newer versions of dependencies, is that older versions may be unavailable at the time it is implemented in another project. This is especially the case with code that isn't published in official repositories. 

## Research

<!--  Current state of IT science being published regadering containers, supply with text mining  -->

To get some insights on this subject, I performed text mining using [Europe PubMed Central](https://europepmc.org). At the time of writing, searching for `github` results in a total of 40.481 articles, of which 36.301 are `Research articles`. Searching `github & docker` results in 1.364 articles, of which 1.198 are Research articles. This would roughly estimate that 3.3% of articles mentioning github, also mention docker. Searching for a similar term `github & container` results in 1.442 articles, of which 1.356 research articles. This is very comparable with the docker search term.


    It seems that containers are under-utilized in bioscience.  


## Solution

<!--  Small tutorial on how to containerize packages (dockerfile, dockerhub) -->

For my most recent project, pySummarizedExperiment (here), I will show the DOCKERFILE used to construct the container. We start by selecting a base image for our code. Choosing a base image depends on the size of the image, but also its dependencies. A barebone image would be very small, but also requires more work to identify the dependencies needed for your code. Luckily, a lot of images have been shared on [Dockerhub](https://hub.docker.com/), so it's best to start there. For this project, I don't need many dependencies, except for Python. I will therefore pick the Python container [here](https://hub.docker.com/_/python). We start by executing the pull command given on Dockerhub `docker pull python`. This makes the image available for us to use. 

Next, we will create a file called `DOCKERFILE` in the project directory. This file starts with a `FROM` statements that specifies the base image to use. We use the `python` image, version 3. 

```dockerfile
FROM python:3


```
# Conclusion 

<!--  Conclusion on containerization, (bio)science should catch up -->

Creating docker containers is easy to do, hard to master. Publishing should be done, even with a simple container to publishing truely reproducible science.
