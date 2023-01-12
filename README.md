# Serverless Web Application - AWS - WildRydes



### Overview

In this Project, I created a simple serverless web application that enables users to request Unicorn Rides from the Wild Rydes fleet. This application presents users with a HTML based interface to select the location of pickup and interfaces on the backend with a REST API to submit the request and dispatch a unicorn. (*just like uber but very simple and static*). The application let's user sign up and create an account to login to the service before requesting a ride.



### Application Architecture

The application uses AWS the following AWS services:

- [AWS Lambda](https://aws.amazon.com/lambda/)

- [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
- [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)
- [Amazon Cognito](https://aws.amazon.com/cognito/)
- [AWS Amplify](https://aws.amazon.com/amplify/) 
- [AWS IAM](https://aws.amazon.com/iam/)



![img](https://d1.awsstatic.com/diagrams/Serverless_Architecture.d930970c77b382db6e0395198aacccd8a27fefb7.png)





### Implementation

* ##### Static Web Hosting

  Used AWS Amplify to host the static web resources of the project such as HTML, CSS, JavaScript and image files which get loaded in the user's browser.

  

  This is the first step I implemented in this project.

  I configured AWS Amplify to host the static resources of the web application with continuous deployment built in. (Continuous deployment - AWS Amplify automatically deploys the web application when it detects any code commit.)

  

  ###### Architecture overview:

  The architecture for this module is straightforward. All of the static web content including HTML, CSS, JavaScript, images, and other files will be managed by AWS Amplify Console. The end users will then access the site using the public website URL exposed by AWS Amplify Console. It is completely serverless, we don't need to run/maintain any web servers or use other services to make the site available.

  ![Architecture Overview](https://d1.awsstatic.com/diagrams/Amplify_Wild_Rydes.1760839c5336d01cd6ac6eabb5d2ad8a37c3304a.png)

  

  - Steps followed: 
    - Selected the nearest region of AWS which provides AWS Amplify service.
    - Created and populated/cloned the git repository with all the HTML, CSS, JS and image files by using AWS CodeCommit.
    - Enabled web hosting using Amplify.
      - Selected **host web app** using **CodeCommit**.
      - Selected the *Repository* and *branch*.
      - Save and Deploy.
    - The process takes a couple of minutes for the Amplify Console to create the necessary resources and deploy the code.







* ##### User management

  Used Amazon Cognito to manage and authenticate user pools to secure the backend API

  In this module, I created an Amazon Cognito user pool to manage users' accounts. Deployed pages that enable the users to register as a new user, verify their email ID and sign in to the site.

  ###### Architecture overview:

  ![Architecture Overview](https://d1.awsstatic.com/Test%20Images/Kate%20Test%20Images/Serverless_Web_App_LP_assets-03.1403870f0fabeb6a11d3e4116092aa6b19b6a924.png)

  When users visit the website they will first register a new user account. After users submit their registration, Amazon Cognito will send a confirmation email with a verification code to the address they provided. To confirm their account, users will return to the site and enter their email address and the verification code they received. After users have a confirmed account (either using the email verification process or a manual confirmation through the console), they will be able to sign in. When users sign in, they enter their username (or email) and password. 

  * Steps followed:
    * Created an Amazon Cognito user pool.
      * Choose Cognito under services.
      * Choose Manage user pools -> Create a User pool.
      * Provide the details, review and hit create pool.
      * Note the pool ID for the newly created user pool.
    * Once the pool is created add an App Client to the pool from the options menu on the left.
    * Note the App Client ID.
    * Update the config.js file in your repository by adding the Pool ID, App Client ID and the AWS region code in the necessary fields.
    * Commit.

  Now the users can sign up, receive the verification email and then successfully login. However, they won't be able to access the site after they click on Login since our API is not yet configured. I configured the API in next step.

​		 





* ##### Serverless Backend

  Used Amazon DynamoDB to provide a persistence layer where data coming from the API's lambda function can be stored.
  
  
  
  ###### Architecture overview:
  
  In this step, I used AWS Lambda and DynamoDB to build a backend process for handling requests for the web application.
  
  ![Architecture Overview](https://d1.awsstatic.com/Test%20Images/Kate%20Test%20Images/Serverless_Web_App_LP_assets_04.76030d61413ff43bd6aa75fbd16e02ad23aec67a.png)
  
  Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table, and then respond to the frontend application with details about the unicorn being dispatched.
  
  The function is invoked from the browser using Amazon API Gateway. However, we will configure the API Gateway in the next module.
  
  * Steps followed for DynamoDB table:
    * Created a DynamoDB table with *RideID* as partition key of *String* type.
    * Noted the ARN of the newly created DynamoDB table as we need it in the upcoming steps.
  
  
  
  
  Every Lambda function has an IAM role associated with it. This role defines what other AWS services the function is allowed to interact with. For the purposes of this project, I created an IAM role that grants my Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to the DynamoDB table.
  
  * Steps followed for IAM:
    * Created a Lambda type role.
    * Permission for the role: *AWSLambdaBasicExecutionRole*.
    * Created an inline policy for the role once the role was created.
    * In the inline policy, selected *PutItem* action and entered the DynamoDB table ARN in under Resources -> Specific -> Table section.
    * Review and create policy.
  
  
  
  Now, I created the Lambda function to run the code in response to events such as an HTTP request. This is a core function that will process API requests from the web application to dispatch a unicorn.
  
  * Steps followed for Lambda:
  
    * Create an *Author from Scratch* lambda function.
    * Selected *Node.js 16.x* for runtime.
    * Chose *existing role* and selected the IAM role created in the previous steps.
    * Create function.
    * Replace the function code with the code in [requestUnicorn.js](https://webapp.serverlessworkshops.io/serverlessbackend/lambda/requestUnicorn.js).
    * Deploy.
  
  * Tested the Lambda function:
  
    * Created a new test event
  
    * Pasted the below code into test event editor.
  
      ```xml
      {    
      			"path": "/ride",    
      			"httpMethod": "POST",    
      			"headers": {        
      					"Accept": "*/*",        
      					"Authorization": "eyJraWQiOiJLTzRVMWZs",        								"content-type": "application/json; charset=UTF-8"    				},    
      			"queryStringParameters": null,    
      			"pathParameters": null,    
      			"requestContext": {        
      					"authorizer": {            
      							"claims": {                
      										"cognito:username": "the_username"            							}        
      					 }    
      			 },    
      			 "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}" 
      }
      ```
  
      
  
    * Save and Test.
  
    * Verified that the execution succeedded and that the function result looks like the following:
  
      ```xml
      {    
      		"statusCode": 201,    
      		"body": "{\"RideId\":\"SvLnijIAtg6inAFUBRT+Fg==\",\"Unicorn\":{\"Name\":\"Rocinante\",\"Color\":\"Yellow\",\"Gender\":\"Female\"},\"Eta\":\"30 seconds\"}",    
      		"headers": {        
      				"Access-Control-Allow-Origin": "*"    
      		 } 
      }
      ```







* ##### RESTful API

  Built a public backend API using AWS Lambda and API Gateway to send and receive data when the javascript is executed in the user's browser.
  
  
  
  ###### Architecture overview:
  
  In this module, I used Amazon API Gateway to expose the Lambda function that I built in the previous module as a RESTful API. This API will be accessible on the public Internet. It will be secured using the Amazon Cognito user pool created in the previous module. 
  
  ![img](https://d1.awsstatic.com/Test%20Images/Kate%20Test%20Images/Serverless_Web_App_LP_assets_05.d1ecdfaab160d7dc00ddbc9e0245fa34b8d8f26b.png)
  
  The static website deployed in the first module already has a page configured to interact with the API that I will build in this module. The page at /ride.html has a simple map-based interface for requesting a unicorn ride. After authenticating using the /signin.html page, the users will be able to select their pickup location by clicking a point on the map and then requesting a ride by choosing the "Request Unicorn" button in the upper right corner.
  
  
  
  * Create a new REST API:
    * From the API Gateway services, chose create API.
    * Select *Build* under *REST API*, choose *Edge optimized* and *Endpoint type*.
    * Create the API.
  * Create Cognito User Pools Authorizer to authenticate API calls.
    * Created a new resource called **/ride** within my API. Then created a **POST method** for that resource and configured it to use a Lambda proxy integration backed by the RequestUnicorn function was created in the first step of this module.
  * Deployed the API using a new stage and noted the **Invoke URL** of the API since we need to put that in the *configure.js* file.
  * Edited the config.js file from the repository and pasted the **Invoke URL** link that I copied in the previous step. This will let the users go forward from the login page and see the map to select a pickup location.
  * Now the final step, verified all the changes by visiting */ride.html* page under the website domain. There, we need to sign in using the ArcGis credentials. After the map has been loaded, I could select a pickup point, **Request Unicorn** and see the unicorn fly to my pickup point.



For the cost purpose, I terminated all my resources.
