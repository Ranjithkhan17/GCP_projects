## Configure Application Load Balancers on GCP cloud ##

Task:
Configure the default region and zone for your resources.
Create an Application Load Balancer.
Test the traffic to your insances.

# Task 1. Set the default region and zone for all resources

```
gcloud config set compute/region Region
gcloud config set compute/zone Zone
```

# Task 2. Create multiple web servers such as www1,www2,www3
```
  gcloud compute instances create www1 \
    --zone=Zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=Debian image ID \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Welcome to RK Techlearning Web Server, Please like this video and Subscribe the channel: This message is from Server1</h3>" | tee /var/www/html/index.html'
```

# Create a firewall rule to allow external traffic to the VM instances(www1,www3,www3) and 
get the external IP addresses of your instances to verify that they are running.
```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

# Task 3. Create an Application Load Balancer
To set up a load balancer with a Compute Engine backend, your VMs need to be in an instance group. 
The managed instance group provides VMs running the backend servers of an external application load balancer
```
gcloud compute instance-templates create lb-backend-template \
   --region=Region \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=Debian image ID \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "<h1> Welcome to RK Techlearning \n This Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
 ```

# Create a managed instance group based on the template:
```
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=Zone
```

# Create the fw-allow-health-check firewall rule.
```
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

#Set up a global static external IP address that your customers use to reach your load balancer

```
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```

#Create a health check for the load balancer:

```
gcloud compute health-checks create http http-basic-check \
  --port 80
```

#Create a backend service add your instance group as the backend to the backend service

```
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```

```
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=Zone \
  --global
  ```
#Create a URL map to route the incoming requests to the default backend service:

```
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service
```

# Create a target HTTP proxy to route requests to your URL map

```
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
```

# Create a global forwarding rule to route incoming requests to the proxy

```
gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80
```


