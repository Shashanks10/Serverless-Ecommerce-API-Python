# AWS Serverless E-Commerce API

A comprehensive serverless e-commerce API built on AWS using Python, Lambda functions, API Gateway, and DynamoDB. This project provides a complete backend solution for user management, product search, shopping cart operations, and order processing.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Infrastructure Setup](#infrastructure-setup)
  - [AWS Account Setup](#1-aws-account-setup)
  - [IAM User and Permissions](#2-iam-user-and-permissions)
  - [AWS CLI Configuration](#3-aws-cli-configuration)
  - [DynamoDB Tables](#4-dynamodb-tables)
  - [Lambda Functions](#5-lambda-functions)
  - [API Gateway](#6-api-gateway)
- [Codebase Setup](#codebase-setup)
  - [Installation](#1-installation)
  - [Project Structure](#2-project-structure)
  - [Configuration](#3-configuration)
  - [Deployment](#4-deployment)
  - [Testing](#5-testing)
- [API Documentation](#api-documentation)
- [Environment Variables](#environment-variables)
- [Troubleshooting](#troubleshooting)

---

## Architecture Overview

This application uses the following AWS services:

- **AWS Lambda**: Serverless compute for running API handlers
- **API Gateway**: RESTful API endpoints
- **DynamoDB**: NoSQL database for storing users, products, cart, and orders
- **IAM**: Role-based access control for Lambda functions

The infrastructure is managed using the **Serverless Framework**, which automates the deployment of all AWS resources.

---

## Prerequisites

Before setting up the infrastructure and codebase, ensure you have:

1. **AWS Account**: An active AWS account with appropriate permissions
2. **Node.js** (v14 or higher): Required for Serverless Framework
3. **Python** (3.10): Runtime for Lambda functions
4. **AWS CLI**: For AWS service interaction
5. **Git**: For version control

---

## Infrastructure Setup

### 1. AWS Account Setup

1. **Create an AWS Account** (if you don't have one)
   - Go to [AWS Sign Up](https://aws.amazon.com/)
   - Complete the registration process
   - Verify your email and payment method

2. **Access AWS Console**
   - Log in to the [AWS Management Console](https://console.aws.amazon.com/)
   - Ensure you have administrative access or sufficient permissions

### 2. IAM User and Permissions

Create an IAM user with programmatic access for the Serverless Framework:

1. **Navigate to IAM Console**
   - Go to IAM → Users → Add users

2. **Create User**
   - Username: `serverless-deploy-user` (or your preferred name)
   - Access type: **Programmatic access**

3. **Attach Policies**
   Attach the following policies (or create a custom policy with these permissions):
   - `AWSLambda_FullAccess`
   - `AmazonAPIGatewayAdministrator`
   - `AmazonDynamoDBFullAccess`
   - `IAMFullAccess` (for creating execution roles)
   - `CloudFormationFullAccess` (Serverless Framework uses CloudFormation)

4. **Save Credentials**
   - Download or copy the **Access Key ID** and **Secret Access Key**
   - Store them securely (you'll need them for AWS CLI configuration)

### 3. AWS CLI Configuration

Configure AWS CLI with your IAM credentials:

```bash
# Install AWS CLI (if not already installed)
# Windows: Download from https://aws.amazon.com/cli/
# macOS: brew install awscli
# Linux: sudo apt-get install awscli

# Configure AWS CLI
aws configure

# Enter the following when prompted:
# AWS Access Key ID: [Your Access Key ID]
# AWS Secret Access Key: [Your Secret Access Key]
# Default region name: us-east-1
# Default output format: json
```

Verify the configuration:

```bash
aws sts get-caller-identity
```

### 4. DynamoDB Tables

The Serverless Framework will automatically create the following DynamoDB tables during deployment:

- **MaxUser** (Users Table)
  - Primary Key: `Email` (String)
  - Billing Mode: Pay-per-request
  - Stores user registration information

- **Products** (Products Table)
  - Primary Key: `ProductId` (String)
  - Billing Mode: Pay-per-request
  - Stores product catalog information

- **Cart** (Cart Table)
  - Primary Key: `UserId` (String) + `ProductId` (String)
  - Billing Mode: Pay-per-request
  - Stores user shopping cart items

- **Orders** (Orders Table)
  - Primary Key: `OrderId` (String)
  - Global Secondary Index: `UserId-index` (for querying orders by user)
  - Billing Mode: Pay-per-request
  - Stores order information

**Note**: These tables are created automatically when you deploy using `serverless deploy`. You don't need to create them manually.

### 5. Lambda Functions

The following Lambda functions are automatically created and configured:

- `registerUser` - User registration handler
- `searchProducts` - Product search handler
- `addToCart` - Add items to cart handler
- `removeFromCart` - Remove items from cart handler
- `checkout` - Order checkout handler
- `getOrderHistory` - Retrieve order history handler
- `trackOrder` - Order tracking handler
- `getUserProfile` - Get user profile handler
- `updateUserProfile` - Update user profile handler

Each Lambda function:
- Runs on Python 3.10 runtime
- Has appropriate IAM permissions for DynamoDB operations
- Is configured with environment variables for table names
- Has a timeout and memory configuration (defaults apply)

### 6. API Gateway

API Gateway is automatically configured to:
- Create RESTful endpoints for each Lambda function
- Enable CORS for cross-origin requests
- Handle request/response transformation
- Provide API endpoint URLs after deployment

**Base URL Format**: `https://[api-id].execute-api.[region].amazonaws.com/[stage]`

---

## Codebase Setup

### 1. Installation

#### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd AWS-Serverless-Ecommerce-API-Handlers-Python
```

#### Step 2: Install Node.js Dependencies

The Serverless Framework requires Node.js. Install it if you haven't already:

```bash
# Check if Node.js is installed
node --version

# If not installed, download from https://nodejs.org/
```

#### Step 3: Install Serverless Framework

Install the Serverless Framework globally:

```bash
npm install -g serverless
```

Verify installation:

```bash
serverless --version
```

#### Step 4: Install Python Dependencies

This project uses Python 3.10. The Lambda functions use the AWS SDK (boto3), which is included in the Lambda runtime environment, so no additional Python packages need to be installed locally unless you're running tests.

However, if you want to install dependencies for local development:

```bash
# Create a virtual environment (recommended)
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Install dependencies (if you have a requirements.txt)
pip install boto3
```

### 2. Project Structure

```
AWS-Serverless-Ecommerce-API-Handlers-Python/
│
├── serverless.yml              # Serverless Framework configuration
├── openapi.json                # OpenAPI/Swagger documentation
├── README.md                   # This file
│
├── user_registration.py       # User registration Lambda handler
├── get_user_profile.py        # Get user profile Lambda handler
├── update_user_profile.py     # Update user profile Lambda handler
│
├── product_search.py          # Product search Lambda handler
│
├── cart.py                    # Add to cart Lambda handler
├── remove_from_cart.py        # Remove from cart Lambda handler
│
├── checkout.py                # Checkout Lambda handler
├── order_history.py           # Get order history Lambda handler
└── order_tracking.py          # Track order Lambda handler
```

**Key Files:**

- **`serverless.yml`**: Defines all AWS resources (Lambda functions, API Gateway, DynamoDB tables, IAM roles)
- **Handler files** (`.py`): Individual Lambda function handlers for each API endpoint
- **`openapi.json`**: API documentation in OpenAPI 3.0 format

### 3. Configuration

#### Environment Variables

The following environment variables are configured in `serverless.yml`:

- `USERS_TABLE_NAME`: `MaxUser`
- `PRODUCTS_TABLE_NAME`: `Products`
- `CART_TABLE_NAME`: `Cart`
- `ORDERS_TABLE_NAME`: `Orders`

These are automatically injected into Lambda functions at runtime.

#### Customizing Configuration

To modify the configuration:

1. **Change AWS Region**: Edit `serverless.yml`:
   ```yaml
   provider:
     region: us-east-1  # Change to your preferred region
   ```

2. **Change Table Names**: Edit the `environment` section in `serverless.yml`:
   ```yaml
   environment:
     USERS_TABLE_NAME: YourTableName
   ```

3. **Modify Lambda Settings**: Add to each function in `serverless.yml`:
   ```yaml
   functions:
     registerUser:
       handler: user_registration.register
       timeout: 30  # seconds
       memorySize: 256  # MB
   ```

### 4. Deployment

#### Deploy to AWS

Deploy all resources (Lambda functions, API Gateway, DynamoDB tables) to AWS:

```bash
# Deploy to default stage (dev)
serverless deploy

# Deploy to specific stage
serverless deploy --stage production

# Deploy with verbose output
serverless deploy --verbose
```

**What happens during deployment:**

1. Serverless Framework packages your Lambda functions
2. Creates/updates CloudFormation stack
3. Creates DynamoDB tables (if they don't exist)
4. Creates/updates Lambda functions
5. Creates/updates API Gateway endpoints
6. Configures IAM roles and permissions

**After deployment**, you'll see output like:

```
Service Information
service: ecommerce-api
stage: dev
region: us-east-1
stack: ecommerce-api-dev
resources: 25
api keys:
  None
endpoints:
  POST - https://xxxxx.execute-api.us-east-1.amazonaws.com/dev/api/register
  GET - https://xxxxx.execute-api.us-east-1.amazonaws.com/dev/api/products
  POST - https://xxxxx.execute-api.us-east-1.amazonaws.com/dev/api/cart/add
  ...
```

**Save the API endpoint URLs** - you'll need them to test the API.

#### Deploy Individual Functions

To deploy a single function (faster for development):

```bash
serverless deploy function -f registerUser
```

#### Remove Deployment

To remove all deployed resources:

```bash
serverless remove
```

**Warning**: This will delete all DynamoDB tables and their data. Make sure to backup data if needed.

### 5. Testing

#### Test API Endpoints

After deployment, test the endpoints using `curl`, Postman, or any HTTP client:

**Example: Register User**

```bash
curl -X POST https://[your-api-url]/dev/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "John Doe",
    "Email": "john@example.com",
    "Password": "password123",
    "Shipping Address": "123 Main St, City, State"
  }'
```

**Example: Search Products**

```bash
curl -X GET "https://[your-api-url]/dev/api/products?Keywords=laptop&Category=Electronics"
```

**Example: Add to Cart**

```bash
curl -X POST https://[your-api-url]/dev/api/cart/add \
  -H "Content-Type: application/json" \
  -d '{
    "UserId": "john@example.com",
    "ProductId": "prod-123"
  }'
```

#### View Logs

Monitor Lambda function logs:

```bash
# View logs for a specific function
serverless logs -f registerUser --tail

# View logs for all functions
serverless logs --tail
```

---

## API Documentation

### Register User API

**Purpose**: Register a new user by storing their details in DynamoDB.

- **HTTP Method**: `POST`
- **Endpoint**: `/api/register`
- **Request Body**:
  ```json
  {
    "Name": "string",
    "Email": "string",
    "Password": "string",
    "Shipping Address": "string"
  }
  ```
- **Features**:
  - Validates required fields
  - Validates email format
  - Hashes the password using SHA-256
  - Checks for duplicate emails and names
  - Enforces password uniqueness and maximum length of 10 characters
  - Returns a success message with user ID or an error if validation fails

### Search Products API

**Purpose**: Search for products based on various criteria.

- **HTTP Method**: `GET`
- **Endpoint**: `/api/products`
- **Query Parameters**:
  - `Keywords`: Search term for product name or description
  - `Category`: Category of the product
  - `Subcategory`: Subcategory of the product
  - `MinPrice`: Minimum price of the product
  - `MaxPrice`: Maximum price of the product
- **Features**:
  - Filters products based on the search criteria
  - Returns a list of matching products or a message if no products are found

### Add to Cart API

**Purpose**: Add a product to a user's cart.

- **HTTP Method**: `POST`
- **Endpoint**: `/api/cart/add`
- **Request Body**:
  ```json
  {
    "UserId": "string",
    "ProductId": "string"
  }
  ```
- **Features**:
  - Validates required fields
  - Checks if the user ID exists in the USERS_TABLE_NAME
  - Checks if the product ID exists in the PRODUCTS_TABLE_NAME
  - Adds the product to the user's cart or increments the quantity if it already exists
  - Returns a success message or an error if validation fails

### Remove from Cart API

**Purpose**: Remove a product from a user's cart.

- **HTTP Method**: `DELETE`
- **Endpoint**: `/api/cart/remove`
- **Request Body**:
  ```json
  {
    "UserId": "string",
    "ProductId": "string"
  }
  ```
- **Features**:
  - Validates required fields
  - Checks if the user ID exists in the USERS_TABLE_NAME
  - Removes the specified product from the user's cart
  - Returns a success message or an error if validation fails

### Checkout API

**Purpose**: Process a user's cart and create an order.

- **HTTP Method**: `POST`
- **Endpoint**: `/api/checkout`
- **Request Body**:
  ```json
  {
    "UserId": "string",
    "ShippingAddress": "string",
    "PaymentMethod": "string",
    "CartItems": [
      {
        "ProductId": "string",
        "Quantity": 1
      }
    ]
  }
  ```
- **Features**:
  - Validates required fields
  - Checks if the user ID exists in the USERS_TABLE_NAME
  - Checks if each product ID in the cart exists in the PRODUCTS_TABLE_NAME
  - Validates the payment method (Credit Card, Debit Card, COD)
  - Calculates the order total and creates a new order in the ORDERS_TABLE_NAME
  - Clears the user's cart after checkout
  - Returns a success message with the order ID or an error if validation fails

### Get Order History API

**Purpose**: Retrieve a user's order history.

- **HTTP Method**: `GET`
- **Endpoint**: `/api/orders`
- **Query Parameters**:
  - `UserID`: The user's ID (email)
- **Features**:
  - Validates required fields
  - Checks if the user ID exists in the USERS_TABLE_NAME
  - Queries the ORDERS_TABLE_NAME for orders associated with the user ID
  - Returns the user's order history or an error if validation fails

### Track Order API

**Purpose**: Get the status of a specific order.

- **HTTP Method**: `GET`
- **Endpoint**: `/api/orders/{orderID}/status`
- **Path Parameters**:
  - `orderID`: The order ID to track
- **Features**:
  - Retrieves order details by order ID
  - Returns order status and information

### Get User Profile API

**Purpose**: Retrieve a user's profile information.

- **HTTP Method**: `GET`
- **Endpoint**: `/api/profile`
- **Query Parameters**:
  - `UserId`: The user's ID (email)
- **Features**:
  - Retrieves user profile from the USERS_TABLE_NAME
  - Returns user information (excluding password)

### Update User Profile API

**Purpose**: Update a user's profile information.

- **HTTP Method**: `PUT`
- **Endpoint**: `/api/profile`
- **Request Body**:
  ```json
  {
    "UserId": "string",
    "updatedProfile": {
      "Name": "string",
      "ShippingAddress": "string"
    }
  }
  ```
- **Features**:
  - Validates required fields
  - Updates user profile in the USERS_TABLE_NAME
  - Returns success message or error

### Common Features Across APIs

- **Error Handling**: Each API has robust error handling to manage client and server errors
- **Validation**: All APIs perform necessary validation to ensure required fields are provided and correctly formatted
- **Logging**: APIs include logging for debugging and monitoring purposes
- **CORS**: All APIs include CORS headers to allow cross-origin requests from permitted domains

---

## Environment Variables

The following environment variables are automatically configured by the Serverless Framework:

| Variable | Value | Description |
|----------|-------|-------------|
| `USERS_TABLE_NAME` | `MaxUser` | DynamoDB table for user data |
| `PRODUCTS_TABLE_NAME` | `Products` | DynamoDB table for product catalog |
| `CART_TABLE_NAME` | `Cart` | DynamoDB table for shopping cart items |
| `ORDERS_TABLE_NAME` | `Orders` | DynamoDB table for order information |

These are accessible in Lambda functions via `os.environ['VARIABLE_NAME']`.

---

## Troubleshooting

### Common Issues

#### 1. AWS Credentials Not Configured

**Error**: `Unable to locate credentials`

**Solution**:
```bash
aws configure
# Enter your Access Key ID and Secret Access Key
```

#### 2. Insufficient IAM Permissions

**Error**: `User is not authorized to perform: lambda:CreateFunction`

**Solution**: Ensure your IAM user has the required policies attached (see [IAM User and Permissions](#2-iam-user-and-permissions))

#### 3. DynamoDB Table Already Exists

**Error**: `Resource already exists`

**Solution**: Either delete the existing table manually from AWS Console or use a different table name in `serverless.yml`

#### 4. Lambda Function Timeout

**Error**: Function times out

**Solution**: Increase timeout in `serverless.yml`:
```yaml
functions:
  registerUser:
    timeout: 30  # Increase from default
```

#### 5. API Gateway CORS Issues

**Error**: CORS policy errors in browser

**Solution**: CORS is already enabled in the configuration. Ensure your frontend is making requests to the correct API endpoint.

#### 6. Deployment Fails

**Error**: CloudFormation stack creation fails

**Solution**:
- Check CloudFormation console for detailed error messages
- Verify all IAM permissions are correct
- Ensure AWS region is correct and accessible
- Check for resource naming conflicts

### Getting Help

- Check AWS CloudWatch Logs for Lambda function errors
- Review Serverless Framework documentation: https://www.serverless.com/framework/docs
- Check AWS service quotas and limits
- Review IAM policy permissions

---

## Additional Resources

- [Serverless Framework Documentation](https://www.serverless.com/framework/docs)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)

---

## License

[Specify your license here]

## Author

[Your name/contact information]
