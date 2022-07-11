---
layout: post
title: Should containers be mandatory in computer (bio)science?
author: "Pascal Maas"
categories: IT, science, publications
image: container.png 
---

<!-- Image of containers being connected -->

<!-- Introduction into containers, chaining them into modules, language independencies -->

Containerization is the concept of creating a shareable virtual environment for your code. This ensures that necessary dependencies are available on any computer, making it able to create reproducible results. 

<!-- Stating the problem when a container is not provided (versioning, out-of-date dependencies, etc. ) -->

A common issue with using published work is the need for version-specific dependencies, that may not be available anymore. 

<!--  Current state of IT science being published regadering containers, supply with text mining  -->

To get some insights on this subject, I performed text mining using europe PubMed Central. It seems that containers are under-utilized in bioscience.  

<!--  Small tutorial on how to containerize packages (dockerfile, dockerhub) -->

For my most recent project, pySummarizedExperiment (here), I will show the DOCKERFILE used to construct the container.  

<!--  Conclusion on containerization, (bio)science should catch up -->

Creating docker containers is easy to do, hard to master. Publishing should be done, even with a simple container to publishing truely reproducible science.