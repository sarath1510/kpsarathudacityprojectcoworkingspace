# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

## A SARATH project 

### Dependencies
#### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster
4. `helm` - apply Helm Charts to a Kubernetes cluster

#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

### Setup


#### 1. Configure a Database
Set up a Postgres database using a Helm Chart.

1. Set up Bitnami Repo
```bash
helm repo add <REPO_NAME> https://charts.bitnami.com/bitnami
```

2. Install PostgreSQL Helm Chart
```
helm install <SERVICE_NAME> <REPO_NAME>/postgresql
```

This should set up a Postgre deployment at `<SERVICE_NAME>-postgresql.default.svc.cluster.local` in your Kubernetes cluster. You can verify it by running `kubectl svc`

By default, it will create a username `postgres`. The password can be retrieved with the following command:
```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default <SERVICE_NAME>-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

echo $POSTGRES_PASSWORD
```

<sup><sub>* The instructions are adapted from [Bitnami's PostgreSQL Helm Chart](https://artifacthub.io/packages/helm/bitnami/postgresql).</sub></sup>

3. Test Database Connection
The database is accessible within the cluster. This means that when you will have some issues connecting to it via your local environment. You can either connect to a pod that has access to the cluster _or_ connect remotely via [`Port Forwarding`](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

* Connecting Via Port Forwarding
```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
```

* Connecting Via a Pod
```bash
kubectl exec -it <POD_NAME> bash
PGPASSWORD="<PASSWORD HERE>" psql postgres://postgres@<SERVICE_NAME>:5432/postgres -c <COMMAND_HERE>
```

4. Run Seed Files
We will need to run the seed files in `db/` in order to create the tables and populate them with data.

```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < <FILE_NAME.sql>
```

### commands executed by me:
AWS:
Ensure the AWS CLI is configured correctly.
aws configure
aws sts get-caller-identity


EKS:creating cluster
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
aws eks --region us-east-1 update-kubeconfig --name my-cluster
kubectl config current-context
kubectl config view

---
eksctl delete cluster --name my-cluster --region us-east-1
---

kubectl get namespace
kubectl get storageclass
kubectl get pods


kubectl exec -it postgresql-77d75d45d5-tpx42 -- bash
psql -U myuser -d mydatabase
\l 
\q

kubectl apply -f pvc.yml 
 kubectl apply -f pv.yml 
 kubectl apply -f postgresql-deployment.yml 
 kubectl apply -f postgresql-service.yml

kubectl get svc
kubectl port-forward service/postgresql-service 5432:5432 &

since i use command prompt:
set DB_PASSWORD=mypassword
set PGPASSWORD=%DB_PASSWORD%
psql --host 127.0.0.1 -U myuser -d mydatabase -p 5432 < 1_create_tables.sql
psql --host 127.0.0.1 -U myuser -d mydatabase -p 5432 < 2_seed_users.sql
psql --host 127.0.0.1 -U myuser -d mydatabase -p 5432 < 3_seed_tokens.sql





### 2. Running the Analytics Application Locally
In the `analytics/` directory:

1. Install dependencies
```bash
pip install -r requirements.txt
```
2. Run the application (see below regarding environment variables)
```bash
<ENV_VARS> python app.py
```

There are multiple ways to set environment variables in a command. They can be set per session by running `export KEY=VAL` in the command line or they can be prepended into your command.

* `DB_USERNAME`
* `DB_PASSWORD`
* `DB_HOST` (defaults to `127.0.0.1`)
* `DB_PORT` (defaults to `5432`)
* `DB_NAME` (defaults to `postgres`)

If we set the environment variables by prepending them, it would look like the following:
```bash
DB_USERNAME=username_here DB_PASSWORD=password_here python app.py
```
The benefit here is that it's explicitly set. However, note that the `DB_PASSWORD` value is now recorded in the session's history in plaintext. There are several ways to work around this including setting environment variables in a file and sourcing them in a terminal session.

3. Verifying The Application
* Generate report for check-ins grouped by dates
`curl <BASE_URL>/api/reports/daily_usage`

* Generate report for check-ins grouped by users
`curl <BASE_URL>/api/reports/user_visits`

### commands executed by me:

pip install -r requirements.txt
set POSTGRES_PASSWORD=mypassword
set DB_USERNAME=myuser
set DB_PASSWORD=%POSTGRES_PASSWORD%
set DB_HOST=127.0.0.1
set DB_PORT=5432
set DB_NAME=mydatabase
python app.py

curl 127.0.0.1:5153/api/reports/daily_usage
curl 127.0.0.1:5153/api/reports/user_visits

## dockerization

docker build -t test-coworking-analytics .
docker run -p 5153:5153 -e DB_HOST=host.docker.internal -e DB_USERNAME=myuser -e DB_PASSWORD=mypassword -e DB_PORT=5432 -e DB_NAME=mydatabase test-coworking-analytics

curl 127.0.0.1:5153/api/reports/daily_usage
curl 127.0.0.1:5153/api/reports/user_visits
--------------
kubectl apply -f ConfigMap.yaml
kubectl apply -f secrets.yaml
kubectl apply -f deployment.yaml


## Project Instructions
1. Set up a Postgres database with a Helm Chart
2. Create a `Dockerfile` for the Python application. Use a base image that is Python-based.
3. Write a simple build pipeline with AWS CodeBuild to build and push a Docker image into AWS ECR
4. Create a service and deployment using Kubernetes configuration files to deploy the application
5. Check AWS CloudWatch for application logs

### Delivered
1. `Dockerfile`
2. Screenshot of AWS CodeBuild pipeline
3. Screenshot of AWS ECR repository for the application's repository
4. Screenshot of `kubectl get svc`
5. Screenshot of `kubectl get pods`
6. Screenshot of `kubectl describe svc coworking-services`
7. Screenshot of `kubectl describe deployment coworking`
8. Screenshot of `kubectl describe svc postgresql-service`
9. Screenshot of `kubectl describe deployment postgresql`
10. All Kubernetes config files used for deployment (ie YAML files)
11. Screenshot of AWS CloudWatch logs for the application
12. `README.md` file in your solution that serves as documentation for your user to detail how your deployment process works and how the user can deploy changes. The details should not simply rehash what you have done on a step by step basis. Instead, it should help an experienced software developer understand the technologies and tools in the build and deploy process as well as provide them insight into how they would release new builds.


### Stand Out Suggestions
Please provide up to 3 sentences for each suggestion. Additional content in your submission from the standout suggestions do _not_ impact the length of your total submission.
1. Specify reasonable Memory and CPU allocation in the Kubernetes deployment configuration:
# ans:
In the Kubernetes deployment file, we allocated 512Mi of memory and 0.5 CPU to the container. This configuration ensures the application has enough resources to handle expected traffic while optimizing the use of cluster resources.
2. In your README, specify what AWS instance type would be best used for the application? Why?
# ans:
For the application, an AWS t3.micro instance is a suitable choice for cost-effectiveness. This instance type offers:

2 vCPUs and 1GB of memory, which is sufficient for lightweight workloads like this one.
Burstable CPU performance to handle occasional spikes in traffic.
3. In your README, provide your thoughts on how we can save on costs?
# ans:
To reduce costs:

Use Spot Instances for non-critical environments, as they offer up to 90% savings compared to On-Demand Instances.
Enable Kubernetes horizontal pod autoscaling, so pods scale up only during high demand and scale down during idle periods.
Use Amazon S3 and ECR lifecycle policies to manage storage costs by automatically deleting older Docker images and logs that are no longer needed.


#testing pull request