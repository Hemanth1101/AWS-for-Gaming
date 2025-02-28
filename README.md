# AWS Architecture for GAMING

This repository contains a complete implementation of a cloud-native SaaS application built on AWS serverless architecture.

## Architecture Overview

This application implements a modern microservices architecture with the following key components:

- **User Authentication**: Secure identity management with Cognito and custom JWT validation
- **API Gateway**: RESTful API with request validation and authorization
- **Serverless Functions**: Lambda functions for business logic
- **Data Storage**: Mix of DynamoDB for high-throughput NoSQL and RDS for relational data
- **Security**: WAF protection, KMS encryption, and IAM-based access control
- **Payment Processing**: Integration with PayPal for subscription management
- **ETL Pipeline**: Data transformation and analytics using AWS Glue
- **Monitoring**: Comprehensive CloudWatch dashboards and alerting

![Architecture Diagram] https://github.com/Hemanth1101/AWS-for-Gaming/blob/main/pubg_architecture.drawio.jpg

## Prerequisites

- AWS CLI configured with appropriate permissions
- Node.js 14+
- Terraform 1.0+
- Docker (for local development)

## Getting Started

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/cloud-native-app.git
   cd cloud-native-app
   ```

2. Install dependencies:
   ```
   npm install
   ```

3. Set up your environment variables:
   ```
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. Deploy the infrastructure:
   ```
   cd infra/terraform
   terraform init
   terraform apply
   ```

5. Deploy the application:
   ```
   npm run deploy
   ```

## Development

### Local Development

Start the local development environment:
```
npm run dev
```

This will:
- Start API Gateway locally using AWS SAM
- Configure local DynamoDB
- Set up local environment variables

### Testing

Run unit tests:
```
npm test
```

Run integration tests:
```
npm run test:integration
```

### Deployment

Deploy to development:
```
npm run deploy:dev
```

Deploy to production:
```
npm run deploy:prod
```

## Documentation

- [API Documentation](docs/api/README.md)
- [User Guide](docs/user-guides/README.md)
- [Architecture Details](docs/architecture/README.md)

## Security

This project implements several security best practices:
- All data is encrypted at rest using KMS
- Secrets are managed via AWS Secrets Manager
- WAF protects against common web vulnerabilities
- JWT-based authentication for all API endpoints

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
