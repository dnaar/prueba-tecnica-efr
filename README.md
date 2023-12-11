# Technical Test Documentation

> efRouting
> 

## [Web page](http://ec2-instances-lb-1910182253.us-east-2.elb.amazonaws.com/)

## Database definition

In order to setup the database required to this project, go to:

> AWS Console > DynamoDB > Tables
> 

![Untitled](Technical%20Test%20Documentation/Untitled.png)

Clicking on ‘Create table’ opens the following panel:

![Untitled](Technical%20Test%20Documentation/Untitled%201.png)

# Data extraction

To create new items in database, it will be implemented a lambda function; therefore, go to:

> AWS Console > Lambda > Functions
> 

![Untitled](Technical%20Test%20Documentation/Untitled%202.png)

Clicking on ‘Create function’ opens the following panel:

![Untitled](Technical%20Test%20Documentation/Untitled%203.png)

On runtime select Python 3.11; since it is going to be accessing DynamoDB resources, an execution role must be defined with the required permissions, refer to [Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html).

Since the python environment does not have the `requests` module, it can be bundled together with the function’s code, refer to [Working with .zip file archives for Python Lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html).

This function will have to be able to request information to [CoinMarketCap API](https://coinmarketcap.com/api/documentation/v1/#operation/getV1CryptocurrencyQuotesLatest) and store it in DynamoDB and retrieve it when an API call is made. To which an [API Gateway](https://aws.amazon.com/es/api-gateway/) will be defined and added as a trigger.

> Source code to all lambda functions defined can be found [here](https://github.com/dnaar/btc-sample-lambda).
> 

# Deploying web page in Docker container

First it will be needed a web page that can receive the data and display it in a chart. For the purpose of this technical test the source code and containerization definitions are available [here](https://github.com/dnaar/btc-prices-sample-frontend).

To deploy this container go to:

> AWS Console > Amazon Elastic Container Service
> 

![Untitled](Technical%20Test%20Documentation/Untitled%204.png)

First it will be needed an Elastic Container Registry (ECR) repository to which will be uploaded the container’s image.

Click on Repositories to go to ECR:

![Untitled](Technical%20Test%20Documentation/Untitled%205.png)

and click on ‘Create repository’:

![Untitled](Technical%20Test%20Documentation/Untitled%206.png)

having done this, the docker image can be pushed to this repository, refer to [Pushing a Docker image](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html); project does this using Github Actions, explained in detail in its corresponding [documentation](https://github.com/dnaar/btc-prices-sample-frontend/blob/main/.github/workflows/README.md#container-deploy-to-ecr).

## After uploading image to ECR

Now, to deploy the container go back to:

> AWS Console > Amazon Elastic Container Service
> 

and click Task definitions:

![Untitled](Technical%20Test%20Documentation/Untitled%207.png)

And click ‘Create new task definition’ and select ‘Create new task definition with JSON’:

![Untitled](Technical%20Test%20Documentation/Untitled%208.png)

This is the JSON structure used to define this project’s task:

```json
{
    "family": "BTC_SAMPLE-TaskDefinition",
    "containerDefinitions": [
        {
            "name": "BTC_SAMPLE-Container",
            "image": "public.ecr.aws/your_id/yourrepo:latest",
            "cpu": 512,
            "memory": 512,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80,
                    "protocol": "tcp"
                }
            ],
            "essential": true,

        }
    ]
}
```

Once the task is defined, it is time to setup a cluster which will run the container’s service. Head to:

> AWS Console > Amazon ECS > Clusters
> 

![Untitled](Technical%20Test%20Documentation/Untitled%209.png)

Click on ‘Create cluster’. This cluster was configured to use ec2 instances. Once the cluster is created, click on it on the list:

![Untitled](Technical%20Test%20Documentation/Untitled%2010.png)

For a new cluster it will have no services. Therefore, to launch one, click on ‘Create’:

![Untitled](Technical%20Test%20Documentation/Untitled%2011.png)

Select the task defined previously and create the service.

> Note: you can setup a load balancer and use its public DNS to access the container. Which is exactly what was done in this project.
> 

# Github Actions

Every repository that has Github Actions implemented has its own documentation with details.

Files:

- [Web application container update](https://github.com/dnaar/btc-prices-sample-frontend/blob/main/.github/workflows/image-deploy.yml)