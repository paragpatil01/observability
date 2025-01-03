# observability
Observability project

🛠️ Installation & Configurations
📦 Step 1: Create EKS Cluster
Prerequisites
Download and Install AWS Cli - Please Refer this link.
Setup and configure AWS CLI using the aws configure command.
Install and configure eksctl using the steps mentioned here.
Install and configure kubectl as mentioned here.
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability

🧰 Step 2: Install kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
🚀 Step 3: Deploy the chart into a new namespace "monitoring"
kubectl create ns monitoring
cd day-2

helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
✅ Step 4: Verify the Installation
kubectl get all -n monitoring
Prometheus UI:
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
NOTE: If you are using an EC2 Instance or Cloud VM, you need to pass --address 0.0.0.0 to the above command. Then you can access the UI on instance-ip:port

Grafana UI: password is prom-operator
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
Alertmanager UI:
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
🧼 Step 5: Clean UP
Uninstall helm chart:
helm uninstall monitoring --namespace monitoring
Delete namespace:
kubectl delete ns monitoring
Delete Cluster & everything else:
eksctl delete cluster --name observability