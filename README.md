# Coworking Space Service

## Getting Started

### Step 1. Create an EKS cluster
```bash
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2

```

### Step 2. Update the Kubeconfig
```bash
aws eks --region us-east-1 update-kubeconfig --name my-cluster
kubectl config current-context

```

### Step 3. Configure ENV variable and Secrets
```bash
kubectl apply -f deployment/configmap.yaml
```

### Step 4. Deploy Database
#### Step 4.1. deploy
```bash
kubectl apply -f deployment/pv.yaml
kubectl apply -f deployment/pvc.yaml
kubectl apply -f deployment/postgresql-deployment.yaml
kubectl apply -f deployment/postgresql-service.yaml

```

#### Step 4.2. seed data
```bash
kubectl port-forward --address 127.0.0.1 service/postgresql-service 5433:5432 &
export DB_PASSWORD=`kubectl get secret dbpassword-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode`
export DB_USER=`kubectl get configMap coworking-config -o jsonpath='{.data.DB_USER}'`
export DB_NAME=`kubectl get configMap coworking-config -o jsonpath='{.data.DB_NAME}'`
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/1_create_tables.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/2_seed_users.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/3_seed_tokens.sql

```

### Step 5. Deploy application
```bash
kubectl apply -f deployment/coworking.yaml
```
get the url and check web app's avaibility

```bash
kubectl get service
#NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)          AGE
#coworking            LoadBalancer   10.100.41.109   ad4f14fbe01ce4081a06570ce460b7d5-987415741.us-east-1.elb.amazonaws.com   5153:31854/TCP   57m
#kubernetes           ClusterIP      10.100.0.1      <none>                                                                   443/TCP          136m
#postgresql-service   ClusterIP      10.100.6.228    <none>                                                                   5432/TCP         118m
```

open this link in browser: http://ad4f14fbe01ce4081a06570ce460b7d5-987415741.us-east-1.elb.amazonaws.com:5153/api/reports/user_visits

### Step 6. Logging
Attach the CloudWatchAgentServerPolicy
```bash
aws iam attach-role-policy \
--role-name eksctl-my-cluster-nodegroup-my-nod-NodeInstanceRole-zWI2AfiNabFJ \
--policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
```

Attach the CloudWatchAgentServerPolicy
```bash
aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name my-cluster
```