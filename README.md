# Create-and-Manage-Cloud-Resources-challenge
Create and Manage Cloud Resources - you will come in with little or no prior cloud knowledge, and come out with practical experience that you can apply to your first Google Cloud project. From writing Cloud Shell commands and deploying your first virtual machine, to running applications on Kubernetes Engine or with load balancing

## Task 1: Create a project "jumphost" VM instance (zone: us-east1-b)

**Solution** 

Navigation menu > Compute engine > VM Instance > Create
* name the instance nucleus-jumphost
* use the machine type of f1-micro
* use the default image type (Debian Linux)

## Task 2: Create a Kubernetes service cluster

**Solution** 

Open Google Cloud Shell

`gcloud container clusters create nucleus-webserver1`

`gcloud container clusters get-credentials nucleus-webserver1`

`kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0`

`kubectl expose deployment hello-app --type=LoadBalancer --port 8080`

`kubectl get service `

## Task 3:Create the web server frontend

**Solution** 

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
* Create an instance template

`gcloud compute instance-templates create nginx-template \`

`--metadata-from-file startup-script=startup.sh`

* Create a target pool

`gcloud compute target-pools create nginx-pool`

* Create a managed instance group

```
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool
```

`gcloud compute instances list`

* Create a firewall rule

`gcloud compute firewall-rules create www-firewall --allow tcp:80`
```
gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool
```

`gcloud compute forwarding-rules list`

* Create a health check

`gcloud compute http-health-checks create http-basic-check`

```
gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80
```

* Create a backend service

```
gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global
```

```
gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global
```
* Create a URL map and target HTTP proxy

```
gcloud compute url-maps create web-map \
--default-service nginx-backend
```

* Create a forwarding rule

`--url-map web-map`

```
gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80
```

`gcloud compute forwarding-rules list`


# now you have to wait 5-10 mins then check your progress :)
