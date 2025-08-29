> Just finished building a secure **AWS VPC project**! 🚀
> - Public **ALB** → Private **EC2 (no public IPs)**
> - Access via **SSM Session Manager** (no bastion or SSH keys)
> - **Gateway VPC Endpoints** (S3 + DynamoDB) to avoid NAT costs
> - Auto Scaling Group with Nginx behind a Load Balancer
>
> This reinforced best practices in **network isolation, IAM/SSM security, and cost optimization**. 
> Repo: vpc-alb-private-ec2-ssm
