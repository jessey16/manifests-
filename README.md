AWS EKS-CLOUD MIGRATION &MONITORING




SUMMARY




This project demonstrates the successful migration of 100MB of critical data from a simulated
legacy environment(on-prem) into a managed Amazon EKS (Kubernetes) cluster in the Mumbai
region. The migration was monitored in real-time using a public Grafana dashboard, providing
transparency and verification of data integrity and system performance.




ARCHITECTURE OVERVIEW




The solution utilizes a "Bridge" architecture to move data securely while maintaining full
observability.

Source: A legacy environment (CloudShell) simulating a local server(on prem)

The Bridge: An S3 Bucket used for staging the migration data.

The Destination: Amazon EKS running in Mumbai.

Monitoring: Prometheus (Data Scraping) and Grafana (Visual Dashboard).










STEP-BY-STEP IMPLEMENTATION




Step 1: Infrastructure Deployment
Provision a production ready cluster using eksctl. This initializes the control plane and two compute
nodes.
eksctl create cluster --name SWOmigration-project --region ap-south-1 --nodegroup-name
standard-nodes --nodes 2 --node-type t3.medium




Step 2: Security & IAM Role Configuration
To allow the Kubernetes nodes to securely pull data from the S3 migration bridge, the
AmazonS3ReadOnlyAccess policy is attached to the Node Group's IAM role.





Identify the Node Role:





ROLE_NAME=$(aws iam list-roles --query 'Roles[?contains(RoleName, "nodegroup")].RoleName'
--output text | tr '\t' '\n' | head -n 1)
Attach Policy:
aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn
arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccessStep 



3: Monitoring Stack Setup





Deploy the Prometheus-Grafana stack via Helm and configure an AWS Load Balancer to provide
a public endpoint for the dashboard.
helm install monitoring prometheus-community/kube-prometheus-stack
kubectl patch svc monitoring-grafana -p '{"spec": {"type": "LoadBalancer"}}'




Step 4: Data Staging (The Bridge)





Generate the migration payload and stage it in a dedicated S3 bucket in the Mumbai region.
head -c 100M </dev/Resh> migration_data.db
aws s3 mb s3://mumbai-migration-project
aws s3 cp migration_data.db s3://mumbai-migration-project/staging/



Step 5: Execution of Migration Job





The migration is executed as a non-restarting Kubernetes Job to ensure data is transferred
exactly once. This uses the specialized AWS CLI image to pull data from the S3 bridge to the
cluster.
kubectl run migration-worker --restart=Never --image=amazon/aws-cli -- \
s3 cp s3://mumbai-migration-project/staging/migration_data.db /tmp/final_destination.db






MONITORING & VISUAL VERIFICATION






The dashboard serves as the "Source of Truth" for verifying the migration.
CPU Utilization: A query monitors the CPU spike during the 100MB transfer, providing proof of
active processing.




Migration Success Indicator: A Grafana "Stat" panel is configured to track the pod phase.



Logic: Once the migration-worker pod reaches the Succeeded phase, the panel turns Green and
displays "MIGRATION SUCCESS".





CONCLUSION
The project successfully met all technical requirements. By integrating IAM security, automated
staging, and real-time observability, demonstrated a robust framework for migrating legacy data
into an Amazon EKS environment.
