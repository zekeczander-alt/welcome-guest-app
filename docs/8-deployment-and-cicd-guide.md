# Deployment and CI/CD Guide Document

**Document Version:** 1.0  
**Date Created:** September 15, 2025  
**Author(s):** AI-Generated  
**Project Name:** Guest Link - A WhatsApp-based Hospitality App

---

## 1. Introduction

### Purpose
This Deployment and CI/CD Guide Document details the processes for deploying Guest Link, a WhatsApp-integrated hospitality management platform. It outlines deployment strategies, environment configurations, and automated pipelines that ensure reliable delivery of features while maintaining security, performance, and compliance requirements. The guide supports Vibe Coding by providing clear deployment paths for AI-generated components and ensuring consistent validation across environments.

### Scope
- Cloud infrastructure setup (AWS and Vercel)
- Environment configuration management
- CI/CD pipeline implementation
- Monitoring and observability setup
- Security and compliance validation
- Database migration handling
- WhatsApp API integration deployment

### Assumptions
- Primary deployment on AWS for scalability and compliance
- Vercel for staging and preview environments
- GitHub Actions for CI/CD automation
- Docker containerization for microservices
- Infrastructure as Code using AWS CDK
- Automated security scanning and compliance checks

---

## 2. Deployment Platforms

### 2.1 AWS (Primary Platform)

#### Core Services
```yaml
Services:
  Compute:
    - ECS Fargate (Microservices)
    - Lambda (Serverless Functions)
  Database:
    - RDS Aurora PostgreSQL (Primary Data)
    - ElastiCache Redis (Caching)
  Storage:
    - S3 (Media Storage)
    - EFS (Shared Files)
  Networking:
    - VPC (Isolated Network)
    - ALB (Load Balancing)
    - CloudFront (CDN)
  Security:
    - WAF (Web Application Firewall)
    - KMS (Key Management)
    - Secrets Manager (Credentials)
```

#### Architecture Diagram
```
                              +----------------+
                              |  CloudFront   |
                              |     (CDN)     |
                              +----------------+
                                     ↓
+----------------+            +----------------+
|   Route 53    |  ------>   |      ALB      |
|    (DNS)      |            | (Load Balancer)|
+----------------+            +----------------+
                                     ↓
                              +----------------+
                              |  ECS Fargate   |
                              | (Microservices)|
                              +----------------+
                                ↙     ↓     ↘
                    +--------+  +----+  +--------+
                    |  RDS   |  |Redis|  |  S3   |
                    |(Aurora)|  |Cache|  |(Media)|
                    +--------+  +----+  +--------+
```

### 2.2 Vercel (Secondary Platform)
- Used for staging environment and PR previews
- Integrated with GitHub for automatic deployments
- Edge functions for lightweight API routes
- Instant rollbacks and deployment monitoring

---

## 3. Environment Setup

### 3.1 Environment Variables
```bash
# Core Configuration
NODE_ENV=production
APP_ENV=production
APP_URL=https://guest-link.app

# Database
DATABASE_URL=postgresql://user:pass@host:5432/guest_link
REDIS_URL=redis://host:6379

# WhatsApp Integration
WHATSAPP_API_URL=https://graph.facebook.com/v17.0
WHATSAPP_API_TOKEN=****
WHATSAPP_WEBHOOK_SECRET=****

# AWS Services
AWS_REGION=us-east-1
AWS_S3_BUCKET=guest-link-media
AWS_CLOUDFRONT_URL=https://d123.cloudfront.net

# Security
JWT_SECRET=****
ENCRYPTION_KEY=****
```

### 3.2 Configuration Files

#### Docker Compose (Local Development)
```yaml
version: '3.8'
services:
  app:
    build: .
    environment:
      - NODE_ENV=development
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: guest_link
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: ****

  redis:
    image: redis:6
    ports:
      - "6379:6379"
```

#### AWS CDK Stack (Production)
```typescript
import * as cdk from 'aws-cdk-lib';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class GuestLinkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // VPC Configuration
    const vpc = new ec2.Vpc(this, 'GuestLinkVPC', {
      maxAzs: 2,
      natGateways: 2
    });

    // ECS Cluster
    const cluster = new ecs.Cluster(this, 'GuestLinkCluster', {
      vpc,
      containerInsights: true
    });

    // Service Definition
    const service = new ecs.FargateService(this, 'GuestLinkService', {
      cluster,
      taskDefinition: taskDefinition,
      desiredCount: 2,
      assignPublicIp: false
    });

    // Add more infrastructure components...
  }
}
```

---

## 4. CI/CD Pipeline

### 4.1 GitHub Actions Workflow

```yaml
name: Guest Link CI/CD
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install Dependencies
        run: npm ci
        
      - name: Lint Code
        run: npm run lint
        
      - name: Type Check
        run: npm run type-check
        
      - name: Run Tests
        run: npm run test:ci
        
      - name: Security Scan
        uses: snyk/actions/node@master
        
  build:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v1
        
      - name: Build Docker Image
        run: docker build -t guest-link .
        
      - name: Push to ECR
        run: |
          aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | \
          docker login --username AWS --password-stdin ${{ secrets.ECR_REGISTRY }}
          docker tag guest-link:latest ${{ secrets.ECR_REGISTRY }}/guest-link:${{ github.sha }}
          docker push ${{ secrets.ECR_REGISTRY }}/guest-link:${{ github.sha }}
          
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster guest-link \
            --service guest-link-service \
            --force-new-deployment
            
      - name: Health Check
        run: |
          ./scripts/health-check.sh \
            --url https://guest-link.app/health \
            --retries 5
```

### 4.2 Deployment Stages

1. **Development**
   - Local environment with Docker Compose
   - PR previews on Vercel
   - Automated tests and linting

2. **Staging**
   - Vercel deployment for frontend
   - AWS staging environment for backend
   - Integration testing and UAT

3. **Production**
   - Blue-green deployment on AWS ECS
   - Database migrations with zero downtime
   - Automated rollback capability

---

## 5. Monitoring and Observability

### 5.1 Monitoring Stack

```typescript
// monitoring/setup.ts
import * as Sentry from '@sentry/nextjs';
import { NewRelic } from 'newrelic';
import { CloudWatch } from 'aws-sdk';

// Sentry Configuration
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Prisma()
  ]
});

// CloudWatch Metrics
const cloudwatch = new CloudWatch();
export const recordMetric = async (name: string, value: number) => {
  await cloudwatch.putMetricData({
    Namespace: 'GuestLink',
    MetricData: [{
      MetricName: name,
      Value: value,
      Unit: 'Count',
      Timestamp: new Date()
    }]
  }).promise();
};
```

### 5.2 Health Checks

```typescript
// api/health/route.ts
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { redis } from '@/lib/redis';
import { whatsapp } from '@/lib/whatsapp';

export async function GET() {
  try {
    // Database Check
    await prisma.$queryRaw`SELECT 1`;
    
    // Redis Check
    await redis.ping();
    
    // WhatsApp API Check
    await whatsapp.checkStatus();
    
    return NextResponse.json({
      status: 'healthy',
      version: process.env.APP_VERSION,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    return NextResponse.json({
      status: 'unhealthy',
      error: error.message
    }, { status: 500 });
  }
}
```

### 5.3 Alert Configuration

```yaml
Alerts:
  HighLatency:
    metric: api_response_time
    threshold: 500ms
    period: 5m
    action: notify_ops_team

  ErrorSpike:
    metric: error_rate
    threshold: 5%
    period: 5m
    action: trigger_incident

  WhatsAppDeliveryDelay:
    metric: message_delivery_time
    threshold: 30s
    period: 5m
    action: notify_ops_team
```

---

## 6. Security and Compliance

### 6.1 Security Measures
- WAF rules for API protection
- Data encryption at rest and in transit
- Regular security scans with Snyk
- JWT token rotation
- Rate limiting implementation

### 6.2 Compliance Checklist
- GDPR data handling verification
- CCPA compliance validation
- Security headers configuration
- Access log retention
- Regular penetration testing

---

## 7. Backup and Recovery

### 7.1 Backup Strategy
```yaml
Databases:
  PostgreSQL:
    type: automated
    frequency: 6h
    retention: 30d
    storage: s3://guest-link-backups

  Redis:
    type: snapshot
    frequency: 1h
    retention: 7d

Media:
  S3:
    type: cross-region-replication
    destination: us-west-2
    versioning: enabled
```

### 7.2 Recovery Procedures
1. Database restoration
2. Service redeployment
3. DNS failover
4. Data consistency verification

---

## 8. Revision History
- September 15, 2025 - v1.0 - Initial deployment and CI/CD guide
- Future updates will be logged here

---

**Document Purpose Recap:** This Deployment and CI/CD Guide Document provides comprehensive instructions for deploying and maintaining Guest Link, ensuring reliable operation while meeting security, performance, and compliance requirements through automated processes and monitoring.