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

To get some insights on this subject, I performed text mining using [Europe PubMed Central](https://europepmc.org). It seems that containers are under-utilized in bioscience.  


## Solution

<!--  Small tutorial on how to containerize packages (dockerfile, dockerhub) -->

For my most recent project, pySummarizedExperiment (here), I will show the DOCKERFILE used to construct the container.  


# Conclusion 

<!--  Conclusion on containerization, (bio)science should catch up -->

Creating docker containers is easy to do, hard to master. Publishing should be done, even with a simple container to publishing truely reproducible science.
