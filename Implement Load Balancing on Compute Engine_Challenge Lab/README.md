# Task 1. Create 3 web servers instances such as web1,web2,web3 
```bash
gcloud compute instances create web1 \
    --zone=europe-west4-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'
```
# Task2: Create Network Load Balancer
	   
1. Set your region and zone environment variables (change if your lab specifie	   
export REGION=europe-west4
export ZONE=europe-west4-a

# 2. Reserve a regional static external IP address
gcloud compute addresses create network-lb-ip-1 --region=$REGION

3. Create an HTTP health check resource
gcloud compute http-health-checks create basic-check

4. Create a target pool in the same region using the health check
gcloud compute target-pools create www-pool \
    --region=$REGION \
    --http-health-check=basic-check

# 5. Add your web server instances to the target pool
	gcloud compute target-pools add-instances www-pool \
		--instances=web1,web2,web3 \
		--instances-zone=$ZONE \
		--region=$REGION

# 6. Create a forwarding rule pointing to the static IP and target pool
gcloud compute forwarding-rules create www-rule \
    --region=$REGION \
    --ports=80 \
    --address=network-lb-ip-1 \
    --target-pool=www-pool
	
	
	
#	Task 3. Create an HTTP load balancer
	************************************
	
# 1. Ensure your region and zone environment variables are set
export REGION=us-west1
export ZONE=us-west1-b

# 2. Create the backend instance template with the startup script and network tag
gcloud compute instance-templates create lb-backend-template \
    --region=$REGION \
    --network=default \
    --subnet=default \
    --tags=allow-health-check \
    --machine-type=e2-medium \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
a2ensite default-ssl
a2enmod ssl
vm_hostname="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
echo "Page served from: $vm_hostname" | tee /var/www/html/index.html
systemctl restart apache2'

# 3. Create the managed instance group (MIG) using the template
gcloud compute instance-groups managed create lb-backend-group \
    --template=lb-backend-template \
    --size=2 \
    --zone=$ZONE

# 4. Create the firewall rule to allow health check traffic from Google Cloud
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80

# 5. Reserve a global static external IPv4 address
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global

# 6. Create the global HTTP health check
gcloud compute health-checks create http http-basic-check \
    --port 80

# 7. Create the backend service
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

# 8. Add the managed instance group to the backend service with a named port
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=$ZONE \
    --global

# 9. Set the named port on the managed instance group
gcloud compute instance-groups managed set-named-ports lb-backend-group \
    --zone=$ZONE \
    --named-ports=http:80

# 10. Create the URL map to route incoming requests to the backend service
gcloud compute url-maps create web-map-http \
    --default-service=web-backend-service

# 11. Create the target HTTP proxy to route requests to your URL map
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map=web-map-http

# 12. Create the global forwarding rule to route external requests to the proxy
gcloud compute forwarding-rules create http-content-rule \
    --load-balancing-scheme=EXTERNAL \
    --network-tier=PREMIUM \
    --address=lb-ipv4-1 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
