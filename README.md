# Cloud Architecture Implementation Guide

## 1. Authentication and Authorization

**Implementation Files:**
- `src/auth/cognito-config.js` - Cognito user pool configuration
- `src/auth/auth-middleware.js` - Authentication middleware
- `infra/terraform/modules/auth/main.tf` - IAM and Cognito infrastructure

**Sample Code:**

```javascript
// src/auth/cognito-config.js
const AWS = require('aws-sdk');
const cognito = new AWS.CognitoIdentityServiceProvider();

const userPoolId = process.env.USER_POOL_ID;
const clientId = process.env.CLIENT_ID;

async function authenticateUser(username, password) {
  const params = {
    AuthFlow: 'USER_PASSWORD_AUTH',
    ClientId: clientId,
    AuthParameters: {
      USERNAME: username,
      PASSWORD: password
    }
  };
  
  try {
    const result = await cognito.initiateAuth(params).promise();
    return {
      token: result.AuthenticationResult.IdToken,
      refreshToken: result.AuthenticationResult.RefreshToken,
      expiresIn: result.AuthenticationResult.ExpiresIn
    };
  } catch (error) {
    console.error('Authentication error:', error);
    throw error;
  }
}

module.exports = { authenticateUser };
```

## 2. API Gateway Setup

**Implementation Files:**
- `infra/terraform/modules/api/main.tf` - API Gateway configuration
- `src/api/routes/index.js` - API route definitions
- `src/api/middleware/validation.js` - Request validation

**Sample Terraform:**

```hcl
# infra/terraform/modules/api/main.tf
resource "aws_api_gateway_rest_api" "main" {
  name        = "${var.project_name}-api"
  description = "Main API Gateway for the application"
  
  endpoint_configuration {
    types = ["REGIONAL"]
  }
}

resource "aws_api_gateway_resource" "users" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  parent_id   = aws_api_gateway_rest_api.main.root_resource_id
  path_part   = "users"
}

resource "aws_api_gateway_method" "users_get" {
  rest_api_id   = aws_api_gateway_rest_api.main.id
  resource_id   = aws_api_gateway_resource.users.id
  http_method   = "GET"
  authorization_type = "COGNITO_USER_POOLS"
  authorizer_id = aws_api_gateway_authorizer.cognito.id
}

resource "aws_api_gateway_integration" "users_get_integration" {
  rest_api_id = aws_api_gateway_rest_api.main.id
  resource_id = aws_api_gateway_resource.users.id
  http_method = aws_api_gateway_method.users_get.http_method
  
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = aws_lambda_function.users_handler.invoke_arn
}
```

## 3. Lambda Functions

**Implementation Files:**
- `src/lambdas/api-handlers/users.js` - User management functions
- `src/lambdas/data-processors/etl.js` - Data transformation
- `src/lambdas/payment-handlers/process-payment.js` - Payment processing

**Sample Lambda Handler:**

```javascript
// src/lambdas/api-handlers/users.js
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    // Process API Gateway event
    const { httpMethod, pathParameters, body } = event;
    
    // GET /users
    if (httpMethod === 'GET' && !pathParameters) {
      const params = {
        TableName: process.env.USERS_TABLE
      };
      
      const result = await dynamoDB.scan(params).promise();
      
      return {
        statusCode: 200,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(result.Items)
      };
    }
    
    // GET /users/{id}
    if (httpMethod === 'GET' && pathParameters && pathParameters.id) {
      const params = {
        TableName: process.env.USERS_TABLE,
        Key: { id: pathParameters.id }
      };
      
      const result = await dynamoDB.get(params).promise();
      
      if (!result.Item) {
        return {
          statusCode: 404,
          body: JSON.stringify({ message: 'User not found' })
        };
      }
      
      return {
        statusCode: 200,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(result.Item)
      };
    }
    
    // POST /users
    if (httpMethod === 'POST') {
      const user = JSON.parse(body);
      user.id = uuidv4();
      user.createdAt = new Date().toISOString();
      
      const params = {
        TableName: process.env.USERS_TABLE,
        Item: user
      };
      
      await dynamoDB.put(params).promise();
      
      return {
        statusCode: 201,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(user)
      };
    }
    
    return {
      statusCode: 400,
      body: JSON.stringify({ message: 'Unsupported method' })
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Internal server error' })
    };
  }
};
```

## 4. Database Configuration

**Implementation Files:**
- `infra/terraform/modules/storage/dynamodb.tf` - DynamoDB tables
- `infra/terraform/modules/storage/rds.tf` - RDS configuration
- `src/data/models/user.js` - Data models

**Sample DynamoDB Configuration:**

```hcl
# infra/terraform/modules/storage/dynamodb.tf
resource "aws_dynamodb_table" "users" {
  name           = "${var.project_prefix}-users"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"
  
  attribute {
    name = "id"
    type = "S"
  }
  
  attribute {
    name = "email"
    type = "S"
  }
  
  global_secondary_index {
    name               = "EmailIndex"
    hash_key           = "email"
    projection_type    = "ALL"
    write_capacity     = 0
    read_capacity      = 0
  }
  
  tags = var.common_tags
}

resource "aws_dynamodb_table" "transactions" {
  name           = "${var.project_prefix}-transactions"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"
  range_key      = "timestamp"
  
  attribute {
    name = "id"
    type = "S"
  }
  
  attribute {
    name = "timestamp"
    type = "S"
  }
  
  attribute {
    name = "userId"
    type = "S"
  }
  
  global_secondary_index {
    name               = "UserTransactions"
    hash_key           = "userId"
    range_key          = "timestamp"
    projection_type    = "ALL"
    write_capacity     = 0
    read_capacity      = 0
  }
  
  tags = var.common_tags
}
```

## 5. Payment Integration

**Implementation Files:**
- `src/lambdas/payment-handlers/paypal.js` - PayPal integration
- `src/api/routes/payments.js` - Payment API endpoints

**Sample PayPal Integration:**

```javascript
// src/lambdas/payment-handlers/paypal.js
const axios = require('axios');

// Get PayPal access token
async function getAccessToken() {
  const clientId = process.env.PAYPAL_CLIENT_ID;
  const clientSecret = process.env.PAYPAL_CLIENT_SECRET;
  const auth = Buffer.from(`${clientId}:${clientSecret}`).toString('base64');
  
  try {
    const response = await axios({
      method: 'post',
      url: `${process.env.PAYPAL_API_URL}/v1/oauth2/token`,
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${auth}`
      },
      data: 'grant_type=client_credentials'
    });
    
    return response.data.access_token;
  } catch (error) {
    console.error('PayPal authentication error:', error);
    throw error;
  }
}

// Create a payment
async function createPayment(amount, currency, description) {
  const accessToken = await getAccessToken();
  
  try {
    const response = await axios({
      method: 'post',
      url: `${process.env.PAYPAL_API_URL}/v2/checkout/orders`,
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`
      },
      data: {
        intent: 'CAPTURE',
        purchase_units: [{
          amount: {
            currency_code: currency,
            value: amount
          },
          description: description
        }]
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('PayPal payment creation error:', error);
    throw error;
  }
}

module.exports = { getAccessToken, createPayment };
```

## 6. Security Configuration

**Implementation Files:**
- `infra/terraform/modules/security/waf.tf` - WAF configuration
- `infra/terraform/modules/security/kms.tf` - Encryption settings
- `src/lambdas/security/log-analyzer.js` - Security monitoring

**Sample WAF Configuration:**

```hcl
# infra/terraform/modules/security/waf.tf
resource "aws_wafv2_web_acl" "main" {
  name        = "${var.project_prefix}-web-acl"
  description = "Web ACL for API protection"
  scope       = "REGIONAL"
  
  default_action {
    allow {}
  }
  
  rule {
    name     = "RateLimit"
    priority = 1
    
    action {
      block {}
    }
    
    statement {
      rate_based_statement {
        limit              = 1000
        aggregate_key_type = "IP"
      }
    }
    
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimit"
      sampled_requests_enabled   = true
    }
  }
  
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 2
    
    override_action {
      none {}
    }
    
    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }
    
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesCommonRuleSet"
      sampled_requests_enabled   = true
    }
  }
  
  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "${var.project_prefix}-web-acl"
    sampled_requests_enabled   = true
  }
}
```

## 7. Data Processing Pipelines

**Implementation Files:**
- `infra/terraform/modules/data/glue.tf` - ETL job configuration
- `src/lambdas/data-processors/transform.js` - Data transformation logic

**Sample ETL Lambda:**

```javascript
// src/lambdas/data-processors/transform.js
const AWS = require('aws-sdk');
const s3 = new AWS.S3();
const dynamoDB = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  try {
    // Get the S3 bucket and key from the event
    const bucket = event.Records[0].s3.bucket.name;
    const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' '));
    
    // Get the file from S3
    const data = await s3.getObject({
      Bucket: bucket,
      Key: key
    }).promise();
    
    // Parse the data (assuming JSON)
    const jsonData = JSON.parse(data.Body.toString('utf-8'));
    
    // Transform the data
    const transformedData = jsonData.map(item => ({
      id: item.id,
      name: item.name,
      email: item.email.toLowerCase(),
      status: item.active ? 'ACTIVE' : 'INACTIVE',
      lastUpdated: new Date().toISOString()
    }));
    
    // Store the transformed data in DynamoDB
    const batchWriteParams = {
      RequestItems: {
        [process.env.USERS_TABLE]: transformedData.map(item => ({
          PutRequest: {
            Item: item
          }
        }))
      }
    };
    
    await dynamoDB.batchWrite(batchWriteParams).promise();
    
    return {
      statusCode: 200,
      body: JSON.stringify({ 
        message: 'Data transformed and loaded successfully',
        count: transformedData.length
      })
    };
  } catch (error) {
    console.error('Error processing data:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Error processing data' })
    };
  }
};
```

## 8. Monitoring and Logging

**Implementation Files:**
- `infra/terraform/modules/monitoring/cloudwatch.tf` - CloudWatch setup
- `src/lambdas/monitoring/alert-handler.js` - Alert processing

**Sample CloudWatch Configuration:**

```hcl
# infra/terraform/modules/monitoring/cloudwatch.tf
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = "${var.project_prefix}-dashboard"
  
  dashboard_body = jsonencode({
    widgets = [
      {
        type   = "metric"
        x      = 0
        y      = 0
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/ApiGateway", "Count", "ApiName", "${var.project_name}-api"]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "API Requests"
        }
      },
      {
        type   = "metric"
        x      = 12
        y      = 0
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/ApiGateway", "4XXError", "ApiName", "${var.project_name}-api"],
            ["AWS/ApiGateway", "5XXError", "ApiName", "${var.project_name}-api"]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "API Errors"
        }
      },
      {
        type   = "metric"
        x      = 0
        y      = 6
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/Lambda", "Invocations", "FunctionName", "${var.project_prefix}-users-handler"],
            ["AWS/Lambda", "Invocations", "FunctionName", "${var.project_prefix}-payment-handler"]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "Lambda Invocations"
        }
      },
      {
        type   = "metric"
        x      = 12
        y      = 6
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/Lambda", "Errors", "FunctionName", "${var.project_prefix}-users-handler"],
            ["AWS/Lambda", "Errors", "FunctionName", "${var.project_prefix}-payment-handler"]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "Lambda Errors"
        }
      }
    ]
  })
}

resource "aws_cloudwatch_log_group" "api_logs" {
  name              = "/aws/apigateway/${var.project_name}-api"
  retention_in_days = 30
}

resource "aws_cloudwatch_metric_alarm" "api_errors" {
  alarm_name          = "${var.project_prefix}-api-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "5XXError"
  namespace           = "AWS/ApiGateway"
  period              = 60
  statistic           = "Sum"
  threshold           = 5
  alarm_description   = "This alarm monitors API Gateway 5XX errors"
  
  dimensions = {
    ApiName = "${var.project_name}-api"
  }
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}
```
