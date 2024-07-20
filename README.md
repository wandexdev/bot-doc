# HNG-DevOps-1(Alpha Bot) Documentation

## Overview

In today's fast-paced development environment, efficient code review and deployment processes are paramount, optimizing processes workflow via automation is key. 
This custom **Alpha bot**, a Node.js application, automates the process of building, deploying and merging pull requests (PRs). It leverages **Docker** for containerization in isolated environments, exposing deployed URL via **Ngrok**, clean up of containers and resources and provides comprehensive feedback on the deployment via comments. It is repository agnostic.

## Architecture
![Bot Architechture](images/alpha-bot.png)
The application listens for pull request events (`opened`, `reopened`, `synchronize`, `closed`) and performs the following actions:
- **Opened/Reopened/Synchronize**: Triggers a deployment of the pull request code using Docker and adds a comment to the pull request.
- **Closed**: Removes the deployed Docker container and its associated resources, and adds a comment to the pull request

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
- services/deploymentService.js: Handles Docker deployment operations
- services/repositoryService.js: Manages interactions with the GitHub API
- index.js: The main application file that sets up the server and handles incoming webhook events.
## Functionality / Features
- Automated Pull Requests Handling: 
- Resource Management:
- Comprehensive Feedback Comments:
- Publicly Accessible Endpoints:
- Custom Domain name:
- Error Handling??

## Getting Started

### Section A: Alpha-Bot Usage
Alpha-Bot is Repository agnostic, it works on any and every respository it is set up for as far as it revolves around containerization of the code.
A few preresquities before running.
- A Github account
- The intended respository to be automated. Docker File  not present, Create [here](https://medium.com/@swalperen3008/what-is-dockerize-and-dockerize-your-project-a-step-by-step-guide-899c48a34df6#:~:text=When%20you%20%E2%80%9CDockerize%E2%80%9D%20an%20application,application%20as%20a%20Docker%20container.)

#### Step 1. Intergration with Intended Repository
- To install the Alpha-bot application, [Click here](https://github.com/apps/alphateam-hng-devops)
![Installation](images/realstepa)
- Next select the repository permissions it should have access to, it then reflects on the applications page
![permissions](images/realstepb)
![view](images/realstepc)


### Section B: Alpha-Bot Testing

### Section C: Alpha-Bot Local Development/SetUp
A local run through of developing the Alpha-Bot from scratch.
Prerequisites for the set up:
- A Github Account
- A Liunx Server preferably Ubuntu
- Basic Javascript Knowledge

#### Step 1. Create a Github App
- At the top right corner of the page, select the github profile photo, scroll down to **settings** and click
![settings](images/stepa)
- Scroll the next drop down and select **developer settings** then **Github Apps** and finally **New Github App**
- The Requirements form pop, fill as follow
![Requirements](stepd.png)
- For the webhook url,insert the url pointing to your ngrok URL or any public URL where the bot is running.
- Webhook secret,

### Step 2. Clone Repository
On your local machine, clone the project
```bash
git clone https://github.com/your-username/github-bot.git
```
```
cd github-bot
```
### Step 3. Install Dependencies
Its a nodejs app, npm package manager
```shell
npm install
```
```
node- --version
```
### Step 4: Code Blocks Explanation
-  `package.json` file manages the projects dependencies and scripts
- `index.js` sets up the Express server and defines the webhook endpoint
```js
import express from 'express';
import bodyParser from 'body-parser';
import 'dotenv/config';
import { addPRComment, verifySignature } from './services/repositoryService.js';
import { triggerDeployment, removeDeployedContainer } from './services/deploymentService.js';
```
- express: Framework for building web servers.
- body-parser: Middleware to parse incoming request bodies.
- dotenv/config: Loads environment variables from a .env file.
- addPRComment, verifySignature: Functions from repositoryService.js.
- triggerDeployment, removeDeployedContainer: Functions from deploymentService.js.
Server Set Up
```js
const app = express();
const port = process.env.PORT || 3003;
app.use(bodyParser.json());
```
- app: Creates an Express application.
- port: Port on which the server will listen.
- app.use(bodyParser.json()): Middleware to parse JSON request bodies.
Webhook Endpoint
```js
app.post('/webhook', async (req, res) => {
    const payload = JSON.stringify(req.body);
    const signature = req.headers['x-hub-signature'];
    console.log(signature);

    if (!verifySignature(payload, signature)) {
        return res.status(400).send('Invalid signature');
    }

    const event = req.headers['x-github-event'];
    console.log(event);
    if (event === 'pull_request') {
        const action = req.body.action;
        console.log(action);
        const prNumber = req.body.number;
        const branchName = req.body.pull_request.head.ref;
        const repoName = req.body.repository.full_name;
        const repoUrl = `https://github.com/${repoName}.git`;
        const repoBranchName = `${repoName.replace('/', '_')}_${branchName.replace('/', '_')}`;
        console.log(repoBranchName);
        const imageName = repoBranchName.toLowerCase();
        const containerName = `${imageName}_pr_${prNumber}`.toLowerCase();

        if (action === 'opened' || action === 'reopened' || action === 'synchronize') {
            try {
                const preDeployMessage = 'The PR is currently being deployed...';
                await addPRComment(repoName, prNumber, preDeployMessage);
                const postDeployMessage = await triggerDeployment(repoUrl, branchName, prNumber, imageName, containerName);
                await addPRComment(repoName, prNumber, postDeployMessage);
                console.log("Deployment completed and comment added!");
            } catch (error) {
                console.error('Error during deployment:', error);
            }
        } else if (action === 'closed') {
            try {
                const message = await removeDeployedContainer(containerName, imageName);
                await addPRComment(repoName, prNumber, message);
                console.log('Container removed and comment added');
            } catch (error) {
                console.error('Error during deployment:', error);
            }
        }
    }

    res.status(200).send('Event received');
});
```
- app.post('/webhook', async (req, res)): Defines the webhook endpoint to handle incoming GitHub events.
- payload: The payload of the incoming webhook event.
- signature: The signature to verify the webhook event's - authenticity.
- event: The type of GitHub event (e.g., pull_request).
- action: The action performed on the pull request (e.g., opened, closed).
- prNumber: The pull request number.
- branchName: The name of the branch associated with the pull request.
-repoName: The full name of the repository.
- repoUrl: The URL of the repository.
- repoBranchName: A unique name for the branch.
- imageName: A lowercase version of the branch name to use as the Docker image name.
- containerName: A name for the Docker container.
`deploymentService.js` file contains function for handling Docker deployments
```js
import { spawn } from 'child_process';
import { promises as fs } from 'fs';
import ngrok from 'ngrok';
```
- spawn: Used to run shell commands.
- fs: Provides file system operations.
- ngrok: Exposes local servers to the internet.

### Step 5. Sort Environment Variables
```env
PORT=3003
WEBHOOK_SECRET=your_github_webhook_secret
APP_ID=your_github_app_id
PRIVATE_KEY_PATH=path_to_your_private_key.pem
INSTALLATION_ID=your_installation_id
NGROK_AUTH_TOKEN=your_ngrok_auth_token
```
- `PORT`: The port number on which the application will run
- `WEBHOOK_SECRET`: The secret token configured in GitHub webhook settings
- `APP_ID`: GitHub App ID.
- `PRIVATE_KEY_PATH`: Path to the private key file generated for your GitHub App.
- `INSTALLATION_ID`: Installation ID of your GitHub App.
- `NGROK_AUTH_TOKEN`: Your ngrok authentication token.

### Step 6. Running the Application
Start the application
```bash
npm start
```
```bash
npm run dev
```
### Step 7. Expose Local Server
```bash
ngrok http 3003
```
## Collaboration Guide

We welcome contributions from the community to improve **Alpha-Bot**! Here's how you can get involved:
- Reporting Issues
- Suggesting Features
- Contributing Code
>1. Fork the repository
>
>2. Create a new branch, 
>
>3. Make your changes
>
>4. Test thoroughly
>
>


## Maintenance

## Troubleshooting

## Licence

