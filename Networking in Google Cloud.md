# Build and Secure Networks in Google Cloud: Challenge Lab


### [GSP322](https://www.cloudskillsboost.google/focuses/12068?locale=en&parent=catalog)

![](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)


## Challenge scenario

You are a security consultant brought in by Jeff, who owns a small local company, to help him with his very successful website (juiceshop). Jeff is new to Google Cloud and had his neighbour's son set up the initial site. The neighbour's son has since had to leave for college, but before leaving, he made sure the site was running.

You need to help out Jeff and perform appropriate configuration for security. Below is the current situation:

![](https://cdn.qwiklabs.com/qEwFTP7%2FkyF3cRwfT3FGObt7L7VLB60%2Bvp92hZVnogw%3D)


## Your challenge

You need to configure this simple environment securely. Your first challenge is to set up appropriate firewall rules and virtual machine tags. You also need to ensure that SSH is only available to the bastion via IAP.

For the firewall rules, make sure:

- The bastion host does not have a public IP address.
- You can only SSH to the bastion and only via IAP.
- You can only SSH to juice-shop via the bastion.
- Only HTTP is open to the world for `juice-shop`.

Tips and tricks:

- Pay close attention to the network tags and the associated VPC firewall rules.
- Be specific and limit the size of the VPC firewall rule source ranges.
- Overly permissive permissions will not be marked correct.

![](https://cdn.qwiklabs.com/BgxgsuLyqMkhxmO3jDlkHE7yGLIR%2B3rrUabKimlgrbo%3D)

Suggested order of actions:

1. Check the firewall rules. Remove the overly permissive rules.

    ```
    gcloud compute firewall-rules delete open-access
    ```

2. Navigate to Compute Engine in the Cloud Console and identify the bastion host. The instance should be stopped. Start the instance.

    ```
    gcloud compute instances start bastion --zone=us-central1-b
    ```

3. The bastion host is the one machine authorized to receive external SSH traffic. Create a firewall rule that allows [SSH (tcp/22) from the IAP service](https://cloud.google.com/iap/docs/using-tcp-forwarding). The firewall rule must be enabled for the bastion host instance using a network tag of `SSH_IAP_NETWORK_TAG`.

    **Create firewall rule**:

    - In the **Cloud Console**, click the **Navigation menu** > **VPC Network** > **Firewall**.
    - Click **Create firewall rule**.
    - Configure the following settings:
        
        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | Name | allow-ssh-iap-ingress |
        | Network | acme-vpc |
        | Direction of traffic | Ingress |
        | Targets | Specified target tags |
        | Target tags | `SSH_IAP_NETWORK_TAG` |
        | Source filter | IP ranges |
        | Source IP ranges | 35.235.240.0/20 |
        | Protocols and ports | Specified protocols and ports, and then *check* tcp, *type:* 22 |
    
    - Click **Create**.

    **Add network tag**:

    - In the **Cloud Console**, click the **Navigation menu** > **Compute Engine** > **VM Instances**.
    - Click on the **bastion host**, and then click **Edit**.
    - Add `SSH_IAP_NETWORK_TAG` to the **Network tags** field.
    - Click **Save**.

4. The `juice-shop` server serves HTTP traffic. Create a firewall rule that allows traffic on HTTP (tcp/80) to any address. The firewall rule must be enabled for the juice-shop instance using a network tag of `HTTP_NETWORK_TAG`.

   **Create firewall rule**:

    - In the **Cloud Console**, click the **Navigation menu** > **VPC Network** > **Firewall**.
    - Click **Create firewall rule**.
    - Configure the following settings:
        
        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | Name | allow-http-ingress |
        | Network | acme-vpc |
        | Direction | Ingress |
        | Targets | Specified target tags |
        | Target tags | `HTTP_NETWORK_TAG` |
        | Source filter | IP ranges |
        | Source IP ranges | 0.0.0.0/0 |
        | Protocols and ports | Specified protocols and ports, and then *check* tcp, *type:* 80 |
    
    - Click **Create**.
    
    **Add network tag**:

    - In the **Cloud Console**, click the **Navigation menu** > **Compute Engine** > **VM Instances**.
    - Click on the **juice-shop host**, and then click **Edit**.
    - Add `HTTP_NETWORK_TAG` to the **Network tags** field.
    - Click **Save**.

5. You need to connect to `juice-shop` from the bastion using SSH. Create a firewall rule that allows traffic on SSH (tcp/22) from `acme-mgmt-subnet` network address. The firewall rule must be enabled for the `juice-shop` instance using a network tag of `SSH_INTERNAL_NETWORK_TAG`.

    **Create firewall rule**.

    - In the **Cloud Console**, click the **Navigation menu** > **VPC Network** > **Firewall**.
    - Click **Create firewall rule**.
    - Configure the following settings:
        
        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | Name | allow-ssh-internal-ingress |
        | Network | acme-vpc |
        | Direction | Ingress |
        | Targets | Specified target tags |
        | Target tags | `SSH_INTERNAL_NETWORK_TAG` |
        | Source filter | IP ranges |
        | Source IP ranges | IP address ranges of acme-mgmt-subnet |
        | Protocols and ports | Specified protocols and ports, and then *check* tcp, *type:* 22 |
    
    - Click **Create**.
    
    **Add network tag**:

    - In the **Cloud Console**, click the **Navigation menu** > **Compute Engine** > **VM Instances**.
    - Click on the **juice-shop host**, and then click **Edit**.
    - Add `SSH_INTERNAL_NETWORK_TAG` to the **Network tags** field.
    - Click **Save**.

6. In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to `juice-shop`.

    ```
    ssh [INTERNAL_IP_OF_JUICE_SHOP]
    ```
