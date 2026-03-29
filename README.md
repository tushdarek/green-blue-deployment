# Blue-Green Deployment with Jenkins and ALB

## Architecture

![](./img/Blue-Green%20deployment%20with%20Jenkins%20pipeline.png)

## Overview

This project implements a **Blue-Green Deployment** for aapplication to achieve **zero-downtime deployments**.  
It uses **Jenkins** for CI/CD and an **AWS Application Load Balancer (ALB)** to switch traffic between environments.


## Features

- Pulls code from GitHub
- Deploys to inactive environment (Blue or Green)
- Performs health check
- Switches ALB traffic automatically
- Rollback if deployment fails

## How It Works

1. Jenkins pulls the latest code from GitHub.
2. Deploys the new version to the inactive environment.
3. Runs health checks.
4. If successful, switches traffic via ALB to the new environment.
5. If health check fails, rollback to previous environment.


### first created two servers
1. green deployment
2. blue deployment

![](./img/project2-server.png)

installed dependencies on both servers

![](./img/project2-webserver.png)

### Step 2: Configure AWS ALB
- Create an **Application Load Balancer (ALB)**.
- Create **two target groups**:
  - `Blue-Target-Group` → points to the Blue server
  - `Green-Target-Group` → points to the Green server
- Configure **health checks** for both target groups.
- Set the ALB to route traffic to the currently active server.

![](./img/project2-TC.png)

![](./img/project2-TG.png)

### Step 3 launch a jenkins server

![](./img/project2-loginjenkins.png)

- Install Jenkins plugins: Git, Pipeline, ssh agent
- install cli in jenkins server
- stored blue & green server (private key) credentials on Jenkins.

![](./img/project2-plugin.png)

## step 4 pushed all required files on github 

index.html
jenkinsfile       

![](./img/project2-githubcode.png)

![](./img/project2githubpush.png)


### Step 5: Set Up Jenkins Pipeline

![](./img/project2-deploycode.png)

Jenkins Pipeline Logic
1. **Pull Code from GitHub**
   - Jenkins fetches the latest code from the repository.
2. **Determine Inactive Server**
   - Check which server (Blue or Green) is currently inactive.
3. **Deploy to Inactive Server**
   - Deploy the new application version to the inactive server.
4. **Health Check**
   - Verify that the new deployment is running correctly.
5. **Switch Traffic via ALB**
   - If health check passes, switch ALB traffic to the new server.
6. **Rollback (if needed)**

   - If health check fails, traffic stays on the current active server.

![](./img/project2-jobsucces.png)   


## Benefits

- Zero-downtime deployments
- Reduced deployment risk
- Quick rollback capability
- Fully automated CI/CD pipeline


## output 

![](./img/project2-DNS.png)

![](./img/project2-blueTGH.png)

## target group switching after new deployment

![](./img/project2-2ndversion.png)

![](./img/project2-greenTGH.png)


Conclusion

Implementing a Blue-Green Deployment pipeline with Jenkins and AWS Application Load Balancer ensures zero-downtime releases and improves the reliability of production deployments. By automatically deploying to an inactive environment, performing health checks, and switching traffic only when the deployment is verified, the system significantly reduces the risk of service disruptions.





