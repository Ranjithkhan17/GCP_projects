# Implement Load Balancing on Compute Engine - Challenge Lab

This guide walks you through implementing load balancing on Google Cloud Platform (GCP) Compute Engine. You'll create multiple web server instances and configure both Network Load Balancing and HTTP Load Balancing.

---

## Table of Contents
1. [Task 1: Create Web Server Instances](#task-1-create-web-server-instances)
2. [Task 2: Configure Network Load Balancer](#task-2-configure-network-load-balancer)
3. [Task 3: Configure HTTP Load Balancer](#task-3-configure-http-load-balancer)

---

## Task 1: Create Web Server Instances

In this task, you'll create three Compute Engine instances (web1, web2, web3) with Apache web servers installed.

### Step 1.1: Create the First Web Server Instance (web1)

This command creates a web server instance with Apache automatically installed and configured to display the server name.

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

### Step 1.2: Create the Second Web Server Instance (web2)

Repeat the process for the second instance. Replace `web1` with `web2` in the instance name and the startup script.

```bash
gcloud compute instances create web2 \
    --zone=europe-west4-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'
```

### Step 1.3: Create the Third Web Server Instance (web3)

Create the third and final instance following the same pattern.

```bash
gcloud compute instances create web3 \
    --zone=europe-west4-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'
```

---

## Task 2: Configure Network Load Balancer

The Network Load Balancer (Layer 4) distributes traffic across your web server instances. This is ideal for non-HTTP protocols and high performance.

### Step 2.1: Set Environment Variables

Define the region and zone for your load balancer configuration. Update these values if your lab specifies a different region.

```bash
export REGION=europe-west4
export ZONE=europe-west4-a
```

### Step 2.2: Reserve a Regional Static External IP Address

This IP address will be assigned to your load balancer and remain constant even if you restart services.

```bash
gcloud compute addresses create network-lb-ip-1 --region=$REGION
```

### Step 2.3: Create an HTTP Health Check Resource

The health check monitors the availability of your web servers. It sends periodic HTTP requests to verify instances are operational.

```bash
gcloud compute http-health-checks create basic-check
```

### Step 2.4: Create a Target Pool

The target pool groups your instances and applies health checks to them. All instances in a pool must be in the same region.

```bash
gcloud compute target-pools create www-pool \
    --region=$REGION \
    --http-health-check=basic-check
```

### Step 2.5: Add Web Server Instances to the Target Pool

Register your three web server instances with the target pool. The load balancer will distribute traffic among these instances.

```bash
gcloud compute target-pools add-instances www-pool \
    --instances=web1,web2,web3 \
    --instances-zone=$ZONE \
    --region=$REGION
```

### Step 2.6: Create a Forwarding Rule

The forwarding rule routes incoming traffic from the static IP address to your target pool on port 80 (HTTP).

```bash
gcloud compute forwarding-rules create www-rule \
    --region=$REGION \
    --ports=80 \
    --address=network-lb-ip-1 \
    --target-pool=www-pool
```

---

## Task 3: Configure HTTP Load Balancer

The HTTP Load Balancer (Layer 7) provides advanced routing capabilities, SSL termination, and content-based routing. This is ideal for web applications.

### Step 3.1: Set Environment Variables

Define the region and zone for your HTTP load balancer. Use a different region if required by your lab.

```bash
export REGION=us-west1
export ZONE=us-west1-b
```

### Step 3.2: Create an Instance Template

An instance template serves as a blueprint for creating managed instances. It includes your startup script to configure Apache web servers.

```bash
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
```

### Step 3.3: Create a Managed Instance Group

A Managed Instance Group (MIG) automatically creates and manages instances based on the template. It simplifies scaling and instance management.

```bash
gcloud compute instance-groups managed create lb-backend-group \
    --template=lb-backend-template \
    --size=2 \
    --zone=$ZONE
```

### Step 3.4: Create a Firewall Rule for Health Checks

Google Cloud health checks originate from specific IP ranges. This firewall rule allows health check traffic to reach your instances.

```bash
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```

### Step 3.5: Reserve a Global Static External IPv4 Address

This global IP address will be assigned to your HTTP load balancer and is accessible from anywhere on the internet.

```bash
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global
```

### Step 3.6: Create a Global HTTP Health Check

This health check verifies that backend instances are healthy by sending periodic HTTP requests. Unhealthy instances are temporarily removed from traffic.

```bash
gcloud compute health-checks create http http-basic-check \
    --port=80
```

### Step 3.7: Create a Backend Service

The backend service defines how traffic is routed to your backend instances, including load balancing policy and health checks.

```bash
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global
```

### Step 3.8: Add the Managed Instance Group to the Backend Service

Register your managed instance group with the backend service. This enables the load balancer to distribute traffic to your instances.

```bash
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=$ZONE \
    --global
```

### Step 3.9: Set Named Port on Managed Instance Group

The named port allows the backend service to communicate with instances on port 80 using a logical name rather than the port number.

```bash
gcloud compute instance-groups managed set-named-ports lb-backend-group \
    --zone=$ZONE \
    --named-ports=http:80
```

### Step 3.10: Create a URL Map

The URL map defines routing rules for your load balancer. You can route different paths to different backend services.

```bash
gcloud compute url-maps create web-map-http \
    --default-service=web-backend-service
```

### Step 3.11: Create an HTTP Proxy

The HTTP proxy acts as an intermediary between clients and your backend service. It handles HTTP requests and forwards them to the appropriate backend.

```bash
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map=web-map-http
```

### Step 3.12: Create a Global Forwarding Rule

The forwarding rule is the final piece. It routes all incoming traffic from your global static IP address to the HTTP proxy on port 80.

```bash
gcloud compute forwarding-rules create http-content-rule \
    --load-balancing-scheme=EXTERNAL \
    --network-tier=PREMIUM \
    --address=lb-ipv4-1 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```

---

## Verification and Testing

After completing all steps, you can verify your load balancers are working:

1. **Get the Network Load Balancer IP:**
   ```bash
   gcloud compute addresses describe network-lb-ip-1 --region=$REGION
   ```

2. **Get the HTTP Load Balancer IP:**
   ```bash
   gcloud compute addresses describe lb-ipv4-1 --global
   ```

3. **Test the load balancer by accessing it in your browser or using curl:**
   ```bash
   curl http://<YOUR_LOAD_BALANCER_IP>
   ```

Refresh multiple times to see traffic being distributed among your different backend instances.

---

## Key Concepts

- **Health Checks:** Monitor instance availability and remove unhealthy instances from traffic rotation
- **Target Pool:** Groups instances for Network Load Balancing (Layer 4)
- **Backend Service:** Groups instances for HTTP/HTTPS Load Balancing (Layer 7)
- **Managed Instance Groups:** Automatically manage instance creation, scaling, and lifecycle
- **Forwarding Rules:** Direct external traffic to your load balancers
- **URL Maps:** Define routing logic for HTTP/HTTPS load balancers

---

## Troubleshooting Tips

- Ensure instances are healthy using: `gcloud compute instance-groups managed list-instances lb-backend-group --zone=$ZONE`
- Check firewall rules: `gcloud compute firewall-rules list`
- Verify backend health: `gcloud compute backend-services get-health web-backend-service --global`
- View load balancer status: `gcloud compute forwarding-rules describe http-content-rule --global`
