# Prometheus_Grafana_Kubernetes_EKS_Monitoring

https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html

https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html

<br>
<br>

## Launch a Kubernetes cluster on AWS EKS (use a Terraform template)

1. Clone this official hashicorp repository with terraform templates:


    ```
    git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
    ```

> This example repository contains configuration to provision a VPC, security groups, and an EKS cluster with the following architecture:
> <img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9ac1f30e-7b2c-4b7c-bcca-95de95b03e04">

<br>
<br>

2. Launch the terraform templates to create infrastructure and a Kubernetes cluster on AWS EKS.

    ```
    terraform init
    terraform validate
    terraform apply
    ```

    <img width="711" alt="Screenshot 2023-05-28 at 17 53 54" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/05c90361-fa22-4b15-8f28-0071bc700691">

<br>
<br>

> If you use AWS Cloud9 as an IDE you also have to disallow AWS Managed Temporary Credentials.
> Go to Cloud9 > Preferences > AWS Settings > AWS Managed Temporary Credentials and turn it off.
> Store your permanent AWS access credentials in the environment. Use ```aws configure``` command.
> 
> <img width="279" alt="Screenshot 2023-05-28 at 14 14 28" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/34f3028e-3fed-4baf-b2af-b1180ea5e5b5">
> <br>
> <img width="584" alt="Screenshot 2023-05-28 at 18 30 32" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9e0c2e9f-1fba-4496-b28f-798ea60070f2">

<br>
<br>

## Launch and configure a master node to manage the Kubernetes cluster 

1. Launch a new EC2 instance or use the current Cloud9 instance. Here is the example of AWS CLI command to launch a t3.small EC2 instance:

    ```
    aws ec2 run-instances --image-id ami-xxxxxxxx --count 1 --instance-type t3.small --key-name MyKeyPair --security-group-ids sg-903004f8 --subnet-id subnet-6e7f829e
    ```

3. Configure it as a master node to work with a cluster named ```education-eks-hCIH6McB```:

    ```
    aws eks update-kubeconfig --name education-eks-hCIH6McB
    ```

    <img width="711" alt="Screenshot 2023-05-28 at 19 42 24" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/1b444954-1063-4f23-90f4-dff928cdc12a">

<br>
<br>

3. Install ```kubectl```

    ```
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
    echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
    kubectl version --short --client
    ```
    <img width="711" alt="Screenshot 2023-05-28 at 19 47 36" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/526e31d2-0d75-4a9a-b6d6-aad35e75c82e">

4. Make sure the master node can access the cluster ```kubectl get svc```

    <img width="491" alt="Screenshot 2023-05-28 at 19 54 05" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/28a52c37-a26f-42fd-8a9c-f33793a482d4">

<br>
<br>

## Install the Kubernetes Metrics Server

The Kubernetes Metrics Server is an aggregator of resource usage data in your cluster, and it is not deployed by default in Amazon EKS clusters. For more information, see Kubernetes Metrics Server on GitHub. The Metrics Server is commonly used by other Kubernetes add ons, such as the ```Horizontal Pod Autoscaler``` or the ```Kubernetes Dashboard```. 

1. Deploy the Metrics Server with the following command:

    ```
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```
2. Verify that the ```metrics-server``` deployment is running the desired number of Pods with the following command.

    ```
    kubectl get deployment metrics-server -n kube-system
    ```
    <img width="711" alt="Screenshot 2023-05-28 at 20 08 04" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/e4efb04d-4152-412f-9afc-62722066a595">

<br> 
<br>

## Install Helm on the master node

Helm does not directly require OpenSSL as a dependency. However, the installation script for Helm uses OpenSSL to verify the integrity of the downloaded binary.

1. Install openssl and helm.

    ```
    sudo yum  install openssl 
    ```

    ```
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

<br> 
<br>

## Deploy Prometheus using Helm

1. Create a Prometheus namespace.

    ```
    kubectl create namespace prometheus
    ```
2. Add the prometheus-community chart repository.

    ```
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```
3. Deploy Prometheus.

    ```
    helm upgrade -i prometheus prometheus-community/prometheus \
        --namespace prometheus \
        --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
    ```
4. Verify that all of the Pods in the prometheus namespace are in the READY state

    ```
    kubectl get pods -n prometheus
    ```
    <img width="711" alt="Screenshot 2023-05-28 at 20 32 05" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/48351a67-ea11-4010-a212-6358f4afef87">

5. Use kubectl to port forward the Prometheus console.

    ```
    kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
    ```

    <img width="711" alt="Screenshot 2023-05-28 at 20 35 12" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/092d168c-2122-4f50-adac-1ba194f8f07d">
    
 <br>
 <br>


## Set up a reverse proxy based on NGINX (to provide the access to Prometheus GUI from the internet)

To access Prometheus running on `127.0.0.1:9090` from the internet, you'll need to set up a reverse proxy or port forwarding to make Prometheus accessible externally.

Here's a general outline of the steps you can follow:

1. Configure a reverse proxy: Set up a reverse proxy server, such as Nginx or Apache, on your EC2 instance. The reverse proxy will receive incoming requests and forward them to the Prometheus server.

2. Install and configure Nginx (example): Install Nginx on your EC2 instance by running the following commands:

   ```
   sudo yum update -y
   sudo yum install nginx
   ```

   Once installed, you can configure Nginx by modifying the configuration file located at `/etc/nginx/nginx.conf`. Here's an example configuration block that you can add to the `http` section of the file:

   ```
   server {
       listen 80;
       server_name YOUR_DOMAIN_OR_IP_ADDRESS;

       location / {
           proxy_pass http://127.0.0.1:9090;
       }
   }
   ```

   Replace `YOUR_DOMAIN_OR_IP_ADDRESS` with the public IP address or domain name of your EC2 instance.
   
    <details markdown=1><summary markdown="span">This is my `/etc/nginx/nginx.conf` file as a reference </summary>

    ```py
    For more information on configuration, see:
    #   * Official English Documentation: http://nginx.org/en/docs/
    #   * Official Russian Documentation: http://nginx.org/ru/docs/

    user nginx;
    worker_processes auto;
    error_log /var/log/nginx/error.log notice;
    pid /run/nginx.pid;

    # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
    include /usr/share/nginx/modules/*.conf;

    events {
        worker_connections 1024;
    }

    http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile            on;
        tcp_nopush          on;
        keepalive_timeout   65;
        types_hash_max_size 4096;

        include             /etc/nginx/mime.types;
        default_type        application/octet-stream;

        # Load modular configuration files from the /etc/nginx/conf.d directory.
        # See http://nginx.org/en/docs/ngx_core_module.html#include
        # for more information.
        include /etc/nginx/conf.d/*.conf;

        server {
            listen       80;
            listen       [::]:80;
            server_name  ec2-3-95-23-122.compute-1.amazonaws.com;
            root         /usr/share/nginx/html;

            location / {
            proxy_pass http://127.0.0.1:9090;
            }

            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;

            error_page 404 /404.html;
            location = /404.html {
            }

            error_page 500 502 503 504 /50x.html;
            location = /50x.html {
            }
        }

    # Settings for a TLS enabled server.
    #
    #    server {
    #        listen       443 ssl http2;
    #        listen       [::]:443 ssl http2;
    #        server_name  _;
    #        root         /usr/share/nginx/html;
    #
    #        ssl_certificate "/etc/pki/nginx/server.crt";
    #        ssl_certificate_key "/etc/pki/nginx/private/server.key";
    #        ssl_session_cache shared:SSL:1m;
    #        ssl_session_timeout  10m;
    #        ssl_ciphers PROFILE=SYSTEM;
    #        ssl_prefer_server_ciphers on;
    #
    #        # Load configuration files for the default server block.
    #        include /etc/nginx/default.d/*.conf;
    #
    #        error_page 404 /404.html;
    #        location = /404.html {
    #        }
    #
    #        error_page 500 502 503 504 /50x.html;
    #        location = /50x.html {
    #        }
    #    }

    }
    ```
    </details>

3. Start Nginx: Start the Nginx service by running the following command:

   ```
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

   Nginx will now listen on port 80 and forward requests to `http://127.0.0.1:9090`.

4. Allow inbound traffic: Ensure that your EC2 instance's security group allows inbound traffic on port 80. You may need to modify the security group settings to allow incoming HTTP traffic.

Once the reverse proxy is set up, you should be able to access Prometheus by navigating to `http://YOUR_DOMAIN_OR_IP_ADDRESS` in a web browser. The reverse proxy will forward the request to Prometheus running on `127.0.0.1:9090` and return the response.

Note: Make sure you have proper security measures in place, such as enabling authentication or restricting access to trusted IP addresses, to protect your Prometheus instance from unauthorized access.

<img width="711" alt="Screenshot 2023-05-28 at 21 57 02" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/db2100f0-053a-46d4-89ee-d18733c4dabe">

<br>
<br>

## Result of setting up Prometheus

Prometheus is now set up to monitor a Kubernetes cluster and scrape metrics

<img width="711" alt="Screenshot 2023-05-29 at 10 06 51" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/92a0f5ad-ec02-49f7-bbf6-a3fc27364420">
<img width="711" alt="Screenshot 2023-05-29 at 10 07 22" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/c992ad4c-46ac-4cef-961d-f4a50d578926">

<img width="711" alt="Screenshot 2023-05-29 at 10 07 47" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/6f1c35c8-4e27-478c-8b8d-cb92a548c2d6">

<img width="711" alt="Screenshot 2023-05-29 at 10 08 13" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/72473a58-17ce-43e7-bb4a-fb67a0e55f54">

<img width="711" alt="Screenshot 2023-05-29 at 10 08 29" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/cb9d246a-8d64-46d5-a0c9-9c2db2871b21">

<br>
<br>

## Install Grafana

1. Copy paste the following in a YAML file, name it grafana.yaml
    ```
    datasources:
      datasources.yaml:
        apiVersion: 1
        datasources:
        - name: Prometheus
          type: prometheus
          url: http://prometheus-server.prometheus.svc.cluster.local
          access: proxy
          isDefault: true
    ```
    This is a configuration file for a monitoring system to visualize and analyze metrics collected by Prometheus. It defines a Prometheus datasource with the following properties:

    - name: The name of the datasource, which is set to "Prometheus".
    - type: The type of the datasource, which is set to "prometheus".
    - url: The URL of the Prometheus server, which is specified as "http://prometheus-server.prometheus.svc.cluster.local". This is the internal URL used to access the Prometheus server within a Kubernetes cluster.
    - access: The access mode for the datasource, which is set to "proxy". This suggests that the datasource should be accessed via a proxy server.
    - isDefault: A boolean flag indicating whether this datasource should be set as the default datasource. In this case, it is set to true, meaning it will be the default datasource.

2.  Grab Grafana Helm charts

    ```
    helm repo add grafana https://grafana.github.io/helm-charts
    ```
    <img width="451" alt="Screenshot 2023-05-29 at 10 56 25" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/42a48fec-9b7f-49ee-a940-a584ed0dc3cd">
    
3. Install Grafana
    ```
    kubectl create namespace grafana
    ```
    <img width="451" alt="Screenshot 2023-05-29 at 10 53 10" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/10b62f1c-c1fd-44fc-9862-39c2889bbcb4">

    ```
    helm install grafana grafana/grafana \
        --namespace grafana \
        --set persistence.storageClassName="gp2" \
        --set persistence.enabled=true \
        --set adminPassword='EKS!sAWSome' \
        --values grafana.yaml \
        --set service.type=LoadBalancer
    ```
    <img width="451" alt="Screenshot 2023-05-29 at 11 36 26" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9aaaae84-c0ac-492a-b4ec-607f594a5f50">

4. Check if Grafana is deployed properly

    ```
    kubectl get all -n grafana
    ```
    <img width="451" alt="Screenshot 2023-05-29 at 11 39 40" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/eb54f43d-9c5d-4239-89a8-6ea251410477">

5. Get Grafana ELB url

    ```
    export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    ```
    ```
    echo "http://$ELB"
    ```
    <img width="451" alt="Screenshot 2023-05-29 at 11 42 29" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/7fc15451-c8c5-438a-887f-445f6543bcd9">

6. Paste the URL into your web browser to access Grafana

    <img width="711" alt="image" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/9ad1b3a0-22af-49ac-8e7d-618e09b33b49">

7. When logging in, use username "admin" and get password by running the following:

    ```
    kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    ```

<br>
<br>

## Set up Grafana to connect to Prometheus for further visualisation and analysis of metrics

   <img width="711" alt="image" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/7e68a31f-cebf-4e5d-8775-cd56880d74fd">

<br>
<br>

1. Grab a Prometheus-server IP

    ```
    kubectl get svc prometheus-server -n prometheus -o jsonpath='http://{.spec.clusterIP}:{.spec.ports[0].port}'
    ```

    <img width="711" alt="Screenshot 2023-05-29 at 12 35 46" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/1ea57b2d-4391-47eb-9fcb-d72c85516301">


2. Add Prometheus as a data source

    <img width="711" alt="Screenshot 2023-05-29 at 12 27 00" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/46411c75-3bde-4555-ba8a-14db4fd3e554">

    <img width="711" alt="Screenshot 2023-05-29 at 12 40 28" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/8cf185ef-e9e1-4768-b9ae-839f7a9e35a7">

3. Create a Dashboard

    <img width="711" alt="Screenshot 2023-05-29 at 14 27 08" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/09b8ed7f-592e-4eef-84f3-cd7cd91ecd3c">

    <img width="711" alt="Screenshot 2023-05-29 at 12 29 30" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/96835451-f2a4-45ab-b52c-33617bf9b182">

    <img width="711" alt="Screenshot 2023-05-29 at 12 46 29" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/d8d975e0-692b-4335-9740-1d2478640777">

    <img width="711" alt="Screenshot 2023-05-29 at 13 01 58" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/ad4e0b9f-9503-4e26-8a37-93b7600f33cb">
    
    
    <img width="711" alt="Screenshot 2023-05-29 at 13 37 19" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/79472d9d-6020-46d8-8c86-5a8b6122734d">
    <img width="711" alt="Screenshot 2023-05-29 at 12 51 57" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/5a5582ad-0859-4794-b9f8-2dda975eeebd">
    
    <img width="711" alt="Screenshot 2023-05-29 at 12 57 25" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/15c880fc-f94c-4fe7-8d3e-997f182c001d">
    
    
    <img width="711" alt="Screenshot 2023-05-29 at 13 36 30" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/969aa485-0c81-4ae0-acf2-d21f0da7d798">
    


<br>
<br>

## Other Grafana dashboards

1. Other Grafana dashboards can be found and retrieved by ID from the official Grafana site available here:

    https://grafana.com/grafana/dashboards/?search=Prometheus

    <img width="711" alt="Screenshot 2023-05-29 at 14 39 30" src="https://github.com/otammato/Prometheus_Grafana_Kubernetes_EKS_Monitoring/assets/104728608/beec9f92-d951-4749-b665-d7c4c8cde4d1">
