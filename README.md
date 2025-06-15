# DevSecOps Application Pipeline

[![Build Status](http://ec2-54-197-89-200.compute-1.amazonaws.com/job/check-code/badge/icon)](http://ec2-54-197-89-200.compute-1.amazonaws.com/job/check-code/)
[![Security Scan](https://img.shields.io/badge/security-scanned-green.svg)](https://github.com/saidrassai/DevSecOps-App)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://hub.docker.com/)

## Project Overview

This is a comprehensive DevSecOps application demonstrating a complete CI/CD pipeline with security integration, multi-environment deployment, and infrastructure as code using Jenkins, Ansible, Docker, and AWS EC2.

### Architecture Components
- **Application**: Go-based fake backend serving static content with MySQL database
- **Frontend**: Battleboat web application (HTML/CSS/JS)
- **Infrastructure**: AWS EC2 instances with Docker containerization
- **CI/CD**: Jenkins pipeline with three environments (dev, preprod, prod)
- **Configuration Management**: Ansible playbooks for automated deployment
- **Security**: Integrated security scanning and vulnerability assessment

## DevSecOps Pipeline

### Three-Environment Strategy

#### 1. Development Environment (`dev`)
- **Purpose**: Development and initial testing
- **Port**: 8080
- **Features**: 
  - Debug logging enabled
  - Database exposed for debugging
  - Rapid deployment for testing

#### 2. Pre-Production Environment (`preprod`)
- **Purpose**: Staging and integration testing
- **Port**: 8081
- **Features**:
  - Production-like configuration
  - Monitoring with Prometheus
  - Security hardened

#### 3. Production Environment (`prod`)
- **Purpose**: Live production deployment
- **Port**: 80 (HTTPS: 443)
- **Features**:
  - High availability with load balancing
  - Full monitoring stack (Prometheus + Grafana)
  - SSL/TLS termination
  - Resource limits and health checks
  - Automated backups

### Jenkins Pipeline Stages

1. **Checkout**: Source code retrieval
2. **Code Quality & Security**: 
   - Golang linting
   - Docker Compose syntax validation
   - Dockerfile security analysis
   - Vulnerability scanning
3. **Unit Testing**: Go test execution with coverage reports
4. **Docker Build**: Multi-stage image building with security scanning
5. **Environment Deployment**: Ansible-based deployment
6. **Post-Deployment Testing**: Health checks and integration tests
7. **Promotion**: Automated promotion between environments

### Security Integration (DevSecOps)

- **Static Code Analysis**: golangci-lint for Go code quality
- **Dockerfile Security**: Hadolint for Dockerfile best practices
- **Image Scanning**: Trivy for container vulnerability assessment
- **Secret Management**: Environment-based secret injection
- **Network Security**: Isolated Docker networks per environment
- **Access Control**: Role-based deployment approvals

## Quick Start

### Prerequisites
- Jenkins server with Docker and Ansible
- AWS EC2 instances for each environment
- Docker registry access
- Ansible inventory configured

### Local Development
```bash
# Clone the repository
git clone https://github.com/saidrassai/DevSecOps-App.git
cd DevSecOps-App

# Run development environment
docker-compose -f docker-compose.dev.yml up -d

# Access the application
curl http://localhost:8080/health
```

### Jenkins Deployment
1. Create three Jenkins jobs: `devsecops-dev`, `devsecops-preprod`, `devsecops-prod`
2. Configure each job with the respective environment parameter
3. Set up automatic triggers:
   - Dev: Trigger on code push
   - Preprod: Manual or successful dev deployment
   - Prod: Manual approval required

## Configuration

### Environment Variables
- `STATIC_PATH`: Static website path (default: `/etc/backend/static`)
- `DATABASE_HOST`: MySQL host (default: `db`)
- `DATABASE_PORT`: MySQL port (default: `3306`)
- `DATABASE_USER`: MySQL username (default: `user`)
- `DATABASE_PASSWORD`: MySQL password (default: `pass`)
- `DATABASE_NAME`: MySQL database name (default: `supinfo`)
- `NODE_ENV`: Environment mode (`development`, `staging`, `production`)
- `LOG_LEVEL`: Logging level (`debug`, `info`, `warn`, `error`)

### Infrastructure Configuration

#### Ansible Inventory Structure
```
ansible/
├── inventory/
│   ├── dev          # Development servers
│   ├── preprod      # Pre-production servers
│   └── prod         # Production servers
└── playbooks/
    └── deploy.yml   # Main deployment playbook
```

#### Docker Compose Files
- `docker-compose.dev.yml`: Development configuration
- `docker-compose.preprod.yml`: Pre-production with monitoring
- `docker-compose.prod.yml`: Production with full stack

## Monitoring and Observability

### Production Monitoring Stack
- **Prometheus**: Metrics collection and alerting
- **Grafana**: Visualization and dashboards
- **Node Exporter**: System metrics
- **Application Metrics**: Custom application health endpoints

### Health Checks
- **Application**: `GET /health` - Returns `200 OK` if database connection is healthy
- **Database**: Connection validation with retry logic
- **Container**: Docker health checks with automatic restart

## Security Features

### Container Security
- Non-root user execution
- Minimal base images
- Regular security updates
- Resource limits and constraints

### Network Security
- Isolated Docker networks per environment
- Service-to-service communication restrictions
- SSL/TLS encryption in production

### Secrets Management
- Environment-specific secret injection
- No hardcoded credentials
- Encrypted at rest and in transit

## Deployment Strategy

### Blue-Green Deployment (Production)
- Zero-downtime deployments
- Instant rollback capability
- Health check validation before traffic switch

### Rolling Updates (Dev/Preprod)
- Gradual instance replacement
- Continuous availability during updates

## Backup and Recovery

### Database Backups
- Automated daily backups
- Point-in-time recovery capability
- Cross-region backup replication

### Application Recovery
- Infrastructure as Code (Ansible)
- Containerized application state
- Rapid environment reconstruction

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Workflow
1. Local development and testing
2. Commit triggers dev pipeline
3. Successful dev deployment enables preprod promotion
4. Manual approval required for production deployment

## Testing

### Unit Tests
```bash
go mod tidy
go test -v ./...
go test -coverprofile=coverage.out ./...
```

### Integration Tests
```bash
# After deployment
curl -f http://your-environment-url/health || exit 1
```

### Security Tests
```bash
# Docker image scanning
trivy image devsecops-app:latest

# Code vulnerability scanning
snyk test
```

## Troubleshooting

### Common Issues

1. **Database Connection Failures**
   - Check database container logs: `docker logs db-container`
   - Verify environment variables
   - Ensure network connectivity

2. **Application Not Starting**
   - Check application logs: `docker logs app-container`
   - Verify port availability
   - Check resource constraints

3. **Pipeline Failures**
   - Review Jenkins console output
   - Verify Ansible inventory connectivity
   - Check Docker registry authentication

### Log Locations
- Application logs: `/opt/devsecops-app/logs/`
- System logs: `/var/log/`
- Container logs: `docker logs <container-name>`

## Architecture Diagrams

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│     DEV     │    │   PREPROD   │    │    PROD     │
│   (8080)    │───▶│   (8081)    │───▶│    (80)     │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Docker    │    │   Docker    │    │   Docker    │
│  Container  │    │  Container  │    │  Swarm      │
└─────────────┘    └─────────────┘    └─────────────┘
```

## License

This project is for educational purposes and private use at SUPINFO International University.

---

**Note**: This project demonstrates DevSecOps best practices and is not intended for production use without additional security hardening and compliance validation.
