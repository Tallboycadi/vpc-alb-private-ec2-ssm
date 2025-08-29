# Runbook — VPC + ALB + EC2 (Private) + SSM + Gateway Endpoints (NO NAT)

This is a **console-first** guide. No Terraform required.

## 1) VPC & Subnets
- VPC: `vpc-portfolio-app` — 10.0.0.0/16
- Public subnets (auto-assign public IP: ON): 10.0.1.0/24 (AZ1), 10.0.2.0/24 (AZ2)
- Private subnets (auto-assign public IP: OFF): 10.0.101.0/24 (AZ1), 10.0.102.0/24 (AZ2)
- Internet Gateway: attach to VPC
- Route tables:
  - Public RT: `0.0.0.0/0 -> igw-...`; associate both public subnets
  - Private RT(s): no default route; associate both private subnets

## 2) Security Groups
- `sg-alb` (ALB): Inbound TCP 80 from 0.0.0.0/0; Outbound all
- `sg-app` (EC2): Inbound TCP 80 from **sg-alb**; Outbound all

## 3) IAM Role for EC2
- Role: `ec2-ssm-portfolio-role`
- Policy: `AmazonSSMManagedInstanceCore`

## 4) Launch Template (Amazon Linux 2023, t3.micro)
- Security group: `sg-app`
- IAM instance profile: `ec2-ssm-portfolio-role`
- User data installs Nginx + page:
```bash
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable nginx
cat >/usr/share/nginx/html/index.html <<'HTML'
<html><head><title>Portfolio App</title></head>
<body style="font-family:Arial;">
  <h1>ALB → Private EC2 (no public IP)</h1>
  <p>If you can read this, the Load Balancer reached a private instance.</p>
</body></html>
HTML
systemctl start nginx
```

## 5) Target Group & ALB
- Target group: HTTP:80, health check path `/`
- ALB: Internet-facing in both public subnets, listener :80 → forward to target group

## 6) Auto Scaling Group (private subnets)
- Desired 2 (min 2, max 3)
- Attach to the target group
- Health checks: EC2 + ELB

## 7) Gateway VPC Endpoints
- S3 (Gateway) → attach to **private** route tables
- DynamoDB (Gateway) → optional, attach to **private** route tables

## 8) Verify
- Open the **ALB DNS name** in your browser; you should see the Nginx page
- Connect via **Session Manager** to an instance and `curl -I http://localhost` → `200 OK`
