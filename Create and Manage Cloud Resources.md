# Create and Manage Cloud Resources: Challenge Lab

### [GSP313](https://www.cloudskillsboost.google/focuses/10258?locale=en&parent=catalog)

![GSP313](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)

## Challenge scenario
You've begun your new role as a Junior Cloud Engineer at Jooli, Inc. Your job is to manage the company's infrastructure. You are expected to have the skills and knowledge necessary to perform these tasks without step-by-step instructions.

Here are some Jooli, Inc. standards to follow:

- Create resources in the default region or zone, unless specified otherwise.
- Use the format "team-resource" for naming. For example, a VM instance could be called **nucleus-webserver1**.
- Allocate cost-effective resource sizes. Be mindful of resource usage, as excessive use can lead to project termination. Use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications like Kubernetes nodes, unless directed otherwise.

## Your challenge

Upon starting your new role, you receive several requests from the Nucleus team. Read the descriptions and create the required resources.

## Task 1. Create project jumphost instance

1. In the **Cloud Console**, go to **Navigation menu** > **Compute Engine** > **VM Instances**.
2. Click **Create instance**.
3. Set the following values (leave all other values as default):

    | Property | Value |
    | -------- | ----- |
    | Name     | `INSTANCE_NAME` |
    | Zone     | us-east1-b |
    | Series   | N1 |
    | Machine type | f1-micro |

4. Click **Create**.

## Task 2. Create Kubernetes service cluster

1. In the **Cloud Console**, click **Activate Cloud Shell** in the top right toolbar.
2. Create a cluster (in the *us-east1-b* zone) to host the service.

    ```
    gcloud config set compute/zone us-east1-b
    gcloud container clusters create nucleus-webserver1
    gcloud container clusters get-credentials nucleus-webserver1
    ```

3. Use the Docker container hello-app (`gcr.io/google-samples/hello-app:2.0`) as a placeholder.

    ```
    kubectl create deployment hello-server \
        --image=gcr.io/google-samples/hello-app:2.0
    ```

4. Expose the app on port `APP_PORT_NUMBER`.

    ```
    kubectl expose deployment hello-server \
        --type=LoadBalancer \
        --port [APP_PORT_NUMBER]
    kubectl get service
    ```

## Task 3. Set up an HTTP load balancer

1. Create a startup-script.

    ```
    cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
    EOF
    ```

2. Create an instance template.

    ```
    gcloud compute instance-templates create nginx-template \
        --metadata-from-file startup-script=startup.sh
    ```

3. Create a target pool.

    ```
    gcloud compute target-pools create nginx-pool \
        --region us-east1
    ```

4. Create a managed instance group.

    ```
    gcloud compute instance-groups managed create nginx-group \
        --base-instance-name nginx \
        --size 2 \
        --template nginx-template \
        --target-pool nginx-pool
    ```

5. Create a firewall rule named `FIREWALL_RULE` to allow traffic (80/tcp).

    ```
    gcloud compute firewall-rules create [FIREWALL_RULE] \
        --allow tcp:80

    gcloud compute forwarding-rules create nginx-lb \
        --region us-east1 \
        --ports=80 \
        --target-pool nginx-pool
    ```

6. Create a health check.

    ```
    gcloud compute http-health-checks create http http-basic-check

    gcloud compute instance-groups managed set-named-ports nginx-group \
        --named-ports http:80
    ```

7. Create a backend service and attach the managed instance group with the named port (http:80).

    ```
    gcloud compute backend-services create nginx-backend \
        --protocol HTTP \
        --http-health-checks http-basic-check \
        --global

    gcloud compute backend-services add-backend nginx-backend \
        --instance-group nginx-group \
        --instance-group-zone us-east1-b \
        --global
    ```

8. Create a URL map and target the HTTP proxy to route requests to your URL map.

    ```
    gcloud compute url-maps create web-map \
        --default-service nginx-backend 

    gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-map
    ```

9. Create a forwarding rule.

    ```
    gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy=http-lb-proxy \
        --ports=80
    ```

10. Test traffic sent to your instances.

    - In the **Cloud Console**, go to **Navigation menu** > **Network services** > **Load balancing**.
    - Click on the load balancer you just created (`web-map`).
    - In the **Backend** section, click on the backend name and confirm that the VMs are **Healthy**. If not, wait a few moments and refresh the page.
    - When the VMs are healthy, test the load balancer using a web browser by navigating to `http://IP_ADDRESS/`, replacing `IP_ADDRESS` with the load balancer's IP address.


