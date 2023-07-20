# Setting Up LocalStack and Testing a Simple GoLang Lambda Function

## Prerequisites

1. **Docker:** LocalStack runs within a Docker container. Visit the Docker website (https://www.docker.com/) and follow the instructions for installing Docker on your operating system.

2. **AWS CLI:** Install the AWS Command Line Interface (CLI) on your machine. You can do this by running the following command in your terminal:

```bash
pip install awscli --upgrade --user
```

## Step 1: Install and Start LocalStack

1. Create a new directory for your LocalStack project, and then create a `docker-compose.yml` file with the following content:

```yaml
version: '3.8'

services:
  localstack:
    image: localstack/localstack
    ports:
      - "4566-4599:4566-4599"
    environment:
      - SERVICES=lambda
      - DEFAULT_REGION=us-east-1
      - LAMBDA_EXECUTOR=docker
      - HOSTNAME=localstack
    volumes:
      - "${TMPDIR:-/tmp/localstack}:/tmp/localstack"

```

2. Save the **docker-compose.yml** file, and in the terminal, navigate to the directory containing the file.

3. Run the following command to start LocalStack:

```bash
docker-compose up
```

LocalStack will start and expose ports 4566-4599 on your localhost for the AWS services.

## Step 2: Set Up AWS CLI
1. Configure the AWS CLI to use LocalStack as the endpoint for the services. In your terminal, run the following command:
```bash
aws configure
```

2. When prompted, enter the following information:
```mathematica
AWS Access Key ID: dummy
AWS Secret Access Key: dummy
Default region name: us-east-1
Default output format: json
```

# Step 3: Create and Test the GoLang Lambda Function

1. Create a file named `main.go` and add the following code:
>  Refer to :  https://github.com/ahmedMHasan/weather-status-aws-lambda-function

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"time"

	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
)

type WeatherResponse struct {
	Weather   string `json:"weather"`
	Timestamp int64  `json:"timestamp"`
}

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	// Retrieve dummy weather information
	weather := "Sunny"

	// Get current timestamp
	timestamp := time.Now().Unix()

	// Create the response object
	response := WeatherResponse{
		Weather:   weather,
		Timestamp: timestamp,
	}

	// Convert the response to JSON
	responseJSON, err := json.Marshal(response)
	if err != nil {
		return events.APIGatewayProxyResponse{StatusCode: http.StatusInternalServerError}, fmt.Errorf("failed to marshal response: %v", err)
	}

	// Return the response
	return events.APIGatewayProxyResponse{
		StatusCode: http.StatusOK,
		Body:       string(responseJSON),
	}, nil
}

func main() {
	lambda.Start(handler)
}
```

2. Package the GoLang Lambda function and deploy it to LocalStack using the AWS CLI:

```bash
GOOS=linux GOARCH=amd64 go build main.go
zip main.zip main
aws --endpoint-url=http://localhost:4566 lambda create-function \
    --function-name my-local-golang-function \
    --runtime go1.x \
    --handler main \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --zip-file fileb://main.zip

```

> Replace **123456789012** in the --role parameter with any arbitrary AWS account number.

3. Invoke the GoLang Lambda function using the AWS CLI:
```bash
aws --endpoint-url=http://localhost:4566 lambda invoke \
--function-name my-local-golang-function \
--payload '{}' \
output.json
```

You should see the output:

```bash
{
"StatusCode": 200,
"ExecutedVersion": "$LATEST"
}
```

To view the function's response, check the output.json file.


> Please note that this is a simplified example, and in a real-world scenario, you might need to set up IAM roles, permissions, and configure other AWS services for your Lambda function to work correctly. The example provided is intended to demonstrate how to set up LocalStack and test a basic GoLang Lambda function locally. For more complex applications, additional configuration and integration with other AWS services would be required.