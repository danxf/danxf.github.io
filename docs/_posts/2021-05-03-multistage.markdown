---
layout: post
title:  "Speeding up deployment lifecycle using Multistaged Docker builds"
date:   2021-05-03 00:02:06 +0200
---
Docker containers provide isolated, replicable and self-contained environments which are most commonly used for running applications. We can use the advantages containers provide, and incorporate Docker’s cache mechanism to create containers not only to help us run but also build applications.

Building software usually comprises of the following steps:
1. Gathering dependencies
2. Static code analysis (compilation)
3. Testing
4. Deploying

Let’s ignore deployment for the sake of simplicity and focus on the first three steps. A typical container which gets dependencies, compiles and tests a java app will look like this:
```
FROM maven-openjdk11
COPY . .
RUN mvn clean deploy
```
Note that a failure in a test suite will result in re-compliation and re-downloading of all dependencies.

Here, Docker’s cache and it’s multi-stage build process can help us. We can breakdown the three steps to intermediate images by utilising `mvn`’s lifecycles. We want an image that only fetches dependencies, one that compiles and one which does the testing. Now if a test fails, and assuming we didn’t change the source code. we can just re-run the test step.

Here’s an example:

```
FROM maven-openjdk11 as dependencies
WORKDIR /workdir
COPY . .
RUN mvn dependency:go-offline
```
```
FROM maven-openjdk11 as compile
WORKDIR /workdir
COPY --from=dependencies /root/.m2 /root/.m2
COPY —from=dependencies /workdir .
RUN mvn clean compile
```
```
FROM maven-openjdk11 as test
WORKDIR /workdir
COPY —from=compile /workdir .
RUN mvn surefire:test
```
