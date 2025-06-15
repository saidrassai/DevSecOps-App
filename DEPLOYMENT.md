# Jenkins Jobs Configuration

## Job 1: DevSecOps-Dev
```groovy
pipeline {
    agent any
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev'],
            description: 'Development Environment'
        )
    }
    triggers {
        githubPush()  // Trigger on every push to main branch
    }
    stages {
        // Full pipeline execution
    }
}
```

## Job 2: DevSecOps-PreProd
```groovy
pipeline {
    agent any
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['preprod'],
            description: 'Pre-Production Environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: true,
            description: 'Skip tests (already passed in dev)'
        )
    }
    triggers {
        upstream(upstreamProjects: 'DevSecOps-Dev', threshold: hudson.model.Result.SUCCESS)
    }
    stages {
        // Deployment focused pipeline
    }
}
```

## Job 3: DevSecOps-Prod
```groovy
pipeline {
    agent any
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['prod'],
            description: 'Production Environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: true,
            description: 'Skip tests (already validated)'
        )
    }
    // Manual trigger only - no automatic triggers for production
    stages {
        // Production deployment with approvals
    }
}
```

## AWS EC2 Infrastructure Setup

### Security Groups
```bash
# Create security group for Dev environment
aws ec2 create-security-group \
    --group-name devsecops-dev-sg \
    --description "DevSecOps Development Security Group"

# Add rules for Dev (ports 22, 8080, 3306)
aws ec2 authorize-security-group-ingress \
    --group-name devsecops-dev-sg \
    --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress \
    --group-name devsecops-dev-sg \
    --protocol tcp --port 8080 --cidr 0.0.0.0/0

# Similar for preprod (8081) and prod (80, 443)
```

### EC2 Instance Launch
```bash
# Launch Dev instances
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 2 \
    --instance-type t3.medium \
    --key-name your-key-pair \
    --security-groups devsecops-dev-sg \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Environment,Value=dev},{Key=Project,Value=DevSecOps}]'
```

## Ansible Configuration

### Update Inventory Files
```ini
# ansible/inventory/dev
[dev]
dev-server-1 ansible_host=YOUR_DEV_IP_1 ansible_user=ec2-user
dev-server-2 ansible_host=YOUR_DEV_IP_2 ansible_user=ec2-user

[dev:vars]
environment=dev
docker_compose_file=docker-compose.dev.yml
```

### SSH Key Setup
```bash
# Copy your SSH keys to Jenkins server
ssh-copy-id -i ~/.ssh/your-key.pem ec2-user@your-dev-server
```

## Environment-Specific Configuration

### Development
- **Purpose**: Rapid development and testing
- **Instances**: 2 x t3.medium
- **Auto-deploy**: On every commit
- **Monitoring**: Basic health checks

### Pre-Production
- **Purpose**: Integration testing and validation
- **Instances**: 2 x t3.large
- **Deploy**: Manual trigger or successful dev
- **Monitoring**: Prometheus + basic alerting

### Production
- **Purpose**: Live application serving
- **Instances**: 3 x t3.xlarge + 2 load balancers
- **Deploy**: Manual approval required
- **Monitoring**: Full stack (Prometheus + Grafana + alerting)
- **Security**: WAF, SSL/TLS, backup automation
