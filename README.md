\# Brain Tasks App – AWS DevOps Deployment



\## 1. Project Overview



Brain Tasks App is a web application deployed as a production-style containerized application on AWS.



The application is:



\* Dockerized using Nginx

\* Stored in Amazon ECR

\* Deployed to Amazon EKS

\* Exposed through a Kubernetes LoadBalancer

\* Automated using AWS CodePipeline and AWS CodeBuild

\* Integrated with GitHub for source control

\* Monitored through Amazon CloudWatch Logs for CodeBuild execution logs



\---



\## 2. Architecture



```text

&#x20;                        GitHub

&#x20;                          |

&#x20;                          v

&#x20;                   AWS CodePipeline

&#x20;                          |

&#x20;                   +------+------+

&#x20;                   |             |

&#x20;                 Source        Build

&#x20;                                 |

&#x20;                                 v

&#x20;                          AWS CodeBuild

&#x20;                                 |

&#x20;                        Docker Image Build

&#x20;                                 |

&#x20;                                 v

&#x20;                        Amazon ECR

&#x20;                                 |

&#x20;                                 v

&#x20;                           Amazon EKS

&#x20;                                 |

&#x20;                      Kubernetes Deployment

&#x20;                        /              \\

&#x20;                   Pod 1               Pod 2

&#x20;                        \\              /

&#x20;                         \\            /

&#x20;                      LoadBalancer Service

&#x20;                                 |

&#x20;                                 v

&#x20;                      Brain Tasks Application

```



\### Pipeline Flow



```text

GitHub

&#x20; ↓

CodePipeline - Source

&#x20; ↓

CodeBuild

&#x20; ↓

Docker Build

&#x20; ↓

Amazon ECR

&#x20; ↓

CodePipeline - Deploy

&#x20; ↓

Amazon EKS

&#x20; ↓

Kubernetes LoadBalancer

&#x20; ↓

Application

```



\---



\## 3. Repository Structure



```text

Brain-Tasks-App/

│

├── dist/

│   ├── index.html

│   ├── vite.svg

│   └── assets/

│

├── k8s/

│   ├── deployment.yaml

│   └── service.yaml

│

├── Dockerfile

├── nginx.conf

├── buildspec.yml

└── README.md

```



\---



\## 4. Dockerization



The application is packaged using Docker and served through Nginx.



\### Dockerfile



The Docker image uses the Nginx Alpine image and copies the production `dist` files into the Nginx web root.



The container exposes port `3000`.



\### Nginx Configuration



Nginx is configured to listen on port `3000` and serve the application.



The configuration also supports SPA routing by redirecting unknown routes to `index.html`.



\### Build Docker Image



```bash

docker build -t brain-tasks-app:latest .

```



\### Run Locally



```bash

docker run -d --name brain-tasks-app-container -p 3000:3000 brain-tasks-app:latest

```



The application can then be accessed at:



```text

http://localhost:3000

```



\---



\## 5. Amazon ECR



An Amazon ECR repository was created in the AWS `eu-north-1` region.



\### ECR Repository



```text

brain-tasks-app

```



\### ECR URI



```text

092454132995.dkr.ecr.eu-north-1.amazonaws.com/brain-tasks-app

```



The Docker image is tagged and pushed to Amazon ECR.



```bash

docker tag brain-tasks-app:latest 092454132995.dkr.ecr.eu-north-1.amazonaws.com/brain-tasks-app:latest

```



```bash

docker push 092454132995.dkr.ecr.eu-north-1.amazonaws.com/brain-tasks-app:latest

```



\---



\## 6. Amazon EKS



An Amazon EKS cluster was created in the `eu-north-1` region.



\### Cluster



```text

brain-tasks-cluster

```



\### Region



```text

eu-north-1

```



The cluster contains two managed worker nodes.



The Kubernetes nodes were verified using:



```bash

kubectl get nodes

```



\---



\## 7. Kubernetes Deployment



The application is deployed using a Kubernetes Deployment with two replicas.



\### Deployment



File:



```text

k8s/deployment.yaml

```



The deployment uses the Docker image stored in Amazon ECR:



```text

092454132995.dkr.ecr.eu-north-1.amazonaws.com/brain-tasks-app:latest

```



The application container listens on port `3000`.



Two application replicas are configured for availability.



Deployment command:



```bash

kubectl apply -f k8s/deployment.yaml

```



Pods can be checked using:



```bash

kubectl get pods

```



\---



\## 8. Kubernetes Service



The application is exposed externally using a Kubernetes LoadBalancer service.



File:



```text

k8s/service.yaml

```



The service:



\* Uses type `LoadBalancer`

\* Exposes port `80`

\* Routes traffic to container port `3000`



Deployment command:



```bash

kubectl apply -f k8s/service.yaml

```



Check the service:



```bash

kubectl get service brain-tasks-app-service

```



The AWS LoadBalancer hostname generated by the service is used to access the application from a web browser.



\---



\## 9. AWS CodeBuild



AWS CodeBuild is used to build and publish the Docker image.



\### CodeBuild Project



```text

brain-tasks-app-build

```



The project uses:



\* Managed CodeBuild image

\* Amazon Linux

\* Standard runtime

\* Docker privileged mode

\* `buildspec.yml`



\### Buildspec



The `buildspec.yml` performs the following steps:



1\. Authenticate with Amazon ECR

2\. Build the Docker image

3\. Tag the image

4\. Push the image to Amazon ECR



The relevant ECR repository is:



```text

092454132995.dkr.ecr.eu-north-1.amazonaws.com/brain-tasks-app

```



\---



\## 10. AWS CodePipeline



AWS CodePipeline automates the application deployment process.



\### Pipeline



```text

brain-tasks-pipeline

```



The pipeline contains three stages:



\### Source



The source repository is GitHub:



```text

https://github.com/EragammaNakkala/Brain-Tasks-App

```



The pipeline monitors the `main` branch.



\### Build



AWS CodeBuild project:



```text

brain-tasks-app-build

```



The build creates the Docker image and pushes it to Amazon ECR.



\### Deploy



The application is deployed to:



```text

Amazon EKS

```



Cluster:



```text

brain-tasks-cluster

```



Deployment method:



```text

Kubectl

```



Kubernetes manifests:



```text

k8s/deployment.yaml

k8s/service.yaml

```



\---



\## 11. CloudWatch Logs



CloudWatch Logs are enabled for the CodeBuild project.



CodeBuild execution logs are available in Amazon CloudWatch Logs under the CodeBuild log group.



This provides visibility into:



\* Build execution

\* Docker image creation

\* ECR authentication

\* ECR image push

\* Build success or failure



\---



\## 12. Application Access



After deployment, the application is exposed through the Kubernetes LoadBalancer.



The service can be checked using:



```bash

kubectl get service brain-tasks-app-service

```



The `EXTERNAL-IP` / LoadBalancer hostname can be opened in a browser.



Example:



```text

http://<load-balancer-hostname>

```



The Brain Tasks application was successfully accessed through the AWS LoadBalancer URL.



\---



\## 13. Deployment Verification



The following commands can be used to verify the deployment.



\### Check Kubernetes Nodes



```bash

kubectl get nodes

```



\### Check Application Pods



```bash

kubectl get pods

```



\### Check Application Service



```bash

kubectl get service brain-tasks-app-service

```



\### Check Application Logs



```bash

kubectl logs deployment/brain-tasks-app

```



\---



\## 14. Technologies Used



\* GitHub

\* Git

\* Docker

\* Nginx

\* Amazon ECR

\* Amazon EKS

\* Kubernetes

\* AWS CodeBuild

\* AWS CodePipeline

\* Amazon CloudWatch Logs

\* AWS CLI

\* kubectl



\---



\## 15. Deployment Result



The final CI/CD pipeline successfully completed the following flow:



```text

GitHub

&#x20;  ↓

AWS CodePipeline

&#x20;  ↓

AWS CodeBuild

&#x20;  ↓

Amazon ECR

&#x20;  ↓

Amazon EKS

&#x20;  ↓

Kubernetes LoadBalancer

&#x20;  ↓

Brain Tasks Application

```



The application was successfully deployed and accessed through the AWS EKS LoadBalancer endpoint.



\---



\## 16. Screenshots



The following screenshots can be included as deployment evidence:



1\. Application running locally on port `3000`

2\. Amazon ECR repository containing the Docker image

3\. EKS cluster and worker nodes

4\. Kubernetes pods running successfully

5\. Kubernetes LoadBalancer service

6\. GitHub repository and source files

7\. CodeBuild project/build result

8\. CodePipeline with Source, Build and Deploy stages successful

9\. CloudWatch CodeBuild logs

10\. Brain Tasks application accessed through the EKS LoadBalancer URL



