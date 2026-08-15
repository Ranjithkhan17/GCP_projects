# Configure Application Load Balancers on GCP

This document shows how to set up an external (HTTP) Application Load Balancer on Google Cloud Platform using Compute Engine backends.

Notes
- Replace placeholders (REGION, ZONE, IMAGE_FAMILY, IMAGE_PROJECT) with values appropriate for your project.
- Commands assume you have the Google Cloud SDK (gcloud) installed and authenticated.

## Prerequisites
- A Google Cloud project with billing enabled
- gcloud installed and authenticated

## Task 1 — Set the default region and zone
Set your default region and zone so subsequent gcloud commands use them.

```bash
gcloud config set compute/region REGION
gcloud config set compute/zone ZONE
```

Replace REGION and ZONE with your desired values, for example `us-central1` and `us-central1-a`.

## Task 2 — Create multiple web server VMs (www1, www2, www3)
Create three VM instances that will serve as backend web servers. Update `--image-family`/`--image-project` as needed.

```bash
gcloud compute instances create www1 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y apache2
    systemctl restart apache2
    echo "<h3>Welcome to RK Techlearning Web Server — This message is from Server1</h3>" | tee /var/www/html/index.html'

gcloud compute instances create www2 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y apache2
    systemctl restart apache2
    echo "<h3>Welcome to RK Techlearning Web Server — This message is from Server2</h3>" | tee /var/www/html/index.html'

gcloud compute instances create www3 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y apache2
    systemctl restart apache2
    echo "<h3>Welcome to RK Techlearning Web Server — This message is from Server3</h3>" | tee /var/www/html/index.html'
```

Create a firewall rule allowing external HTTP (port 80) to the instances tagged with `network-lb-tag` and verify external IPs.

```bash
gcloud compute firewall-rules create www-firewall-network-lb \
  --target-tags=network-lb-tag \
  --allow=tcp:80

# Get external IPs
gcloud compute instances list --filter="name~'www[1-3]'" --format="table(name,EXTERNAL_IP)"
```

## Task 3 — Create a managed instance group (backend) and load balancer
To use a Compute Engine backend with an external load balancer, place your VMs in a managed instance group created from an instance template.

### 3.1 Create an instance template
Adjust the startup script as needed.

```bash
gcloud compute instance-templates create lb-backend-template \
  --network=default \
  --subnet=default \
  --tags=allow-health-check \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y apache2
    a2ensite default-ssl || true
    a2enmod ssl || true
    vm_hostname="$(curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
    echo "<h1>Welcome to RK Techlearning</h1><p>This page served from: $vm_hostname</p>" | tee /var/www/html/index.html
    systemctl restart apache2'
```

### 3.2 Create a managed instance group
Create a managed instance group based on the template. Use a zone (not region) for zonal MIGs.

```bash
gcloud compute instance-groups managed create lb-backend-group \
  --template=lb-backend-template \
  --size=2 \
  --zone=ZONE
```

### 3.3 Allow health checks
Create a firewall rule to allow health check probes from Google's health check ranges.

```bash
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

### 3.4 Reserve a global static IP for the load balancer

```bash
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```

### 3.5 Create a health check

```bash
gcloud compute health-checks create http http-basic-check \
  --port 80
```

### 3.6 Create a backend service and add the instance group

```bash
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=ZONE \
  --global
```

### 3.7 Create a URL map

```bash
gcloud compute url-maps create web-map-http \
  --default-service web-backend-service
```

### 3.8 Create a target HTTP proxy

```bash
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map-http
```

### 3.9 Create a global forwarding rule

```bash
gcloud compute forwarding-rules create http-content-rule \
  --address=lb-ipv4-1 \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80
```

## Testing
- Visit the reserved global IP (lb-ipv4-1) in a browser to confirm traffic is served by the backend instances.
- Use `gcloud compute backend-services get-health web-backend-service --global` to view backend health.

## Cleanup
To avoid charges, delete resources when finished (instance-groups, templates, instances, firewall rules, backend service, forwarding rule, health checks, reserved IP).

```bash
# Example: delete the forwarding rule and IP
gcloud compute forwarding-rules delete http-content-rule --global
gcloud compute addresses delete lb-ipv4-1 --global
```
