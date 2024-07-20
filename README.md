# HNG-DevOps-1(Alpha Bot) Intergration Guide

## Overview

In today's fast-paced development environment, efficient code review and deployment processes are paramount, optimizing processes workflow via automation is key. 
This **Alpha bot** automates the process of building, deploying and merging pull requests (PRs). It leverages **Docker** for containerization in isolated environments, exposing deployed URL via **Ngrok**, clean up of containers and resources and provides comprehensive feedback on the deployment via comments.

## Architecture
[Bot Architechture](https://excalidraw.com/#json=c7hLjaZqz4cWcCxWJDAUY,7umdP-IFd03OQu4hUovwiQ)

## Folder Structure
```
|--- services
|    |--- deploymentService.js
|    |--- repositoryService.js
|--- .gitignore
|--- index.js
|--- package.json
|--- README.md
```

## Getting Started

W
