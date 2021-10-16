# cicdworkflowserverless
# Building CI/CD workflows for Serverless Applications

This lab is provided as part of **[AWS Innovate - Modern Applications Edition](https://aws.amazon.com/events/aws-innovate/modern-apps/)**, click [here](https://google.com) to explore the full list of hands-on labs.

ℹ️ You will run this lab in your own AWS account. Please follow directions at the end of the lab to remove resources to minimize costs.

This lab walks you through creating a CI/CD workflow for serveress applications. We will start with some simple workflows progressively building more complex workflows.

## Content 


## Getting started
In software engineering, **CI/CD** or **CICD** is the combined practices of continuous integration (CI) and either continuous delivery or continuous deployment (CD).

CI/CD bridges the gaps between development and operation activities and teams by enforcing automation in building, testing and deployment of applications. The process contrasts with traditional methods where all updates were integrated into one large batch before rolling out the newer version.

**Serverless applications** feature automatic scaling, built-in high availability, and a pay-for-use billing model to increase agility and optimize costs. These technologies also eliminate infrastructure management tasks like capacity provisioning and patching, so you can focus on writing code that serves your customers. Serverless applications start with AWS Lambda, an event-driven compute service natively integrated with over 200 AWS services and software as a service (SaaS) applications.

### Step 1. Deploying a lambda application
1. On the AWS Console navigate to the AWS Lambda service and select the **Applications** option on the left pane.
2. Select Create application and select the **Serverless API backend** option.
serverlessapibackendapp.png
3. This will give us a RESTful web API that uses DynamoDB to manage state. Click on Next.
4. Enter a name for the application as ``serverless-start``
5. Select runtime as Node.js 14.x
6. We will use [AWS SAM](https://aws.amazon.com/serverless/sam/) for this application. Leave the Template format as AWS SAM(YAML).
7. In the Source control section we will use CodeStar Connections to create a private repository in your GitHub account. Select ``Create new connection`` and click on Connect with CodeStar Connections.
    ![awsconnectorgithub](/images/awsconnectorgithub.png)
    ![githubinstallapp](/images/githubinstallapp.png)
    ![installawsconnecttorgithub](/images/installawsconnecttorgithub.png)
    ![connecttogithub](/images/connecttogithub.png)
    ![sourcecontrolserverlessstart](/images/sourcecontrolserverlessstart.png)

8. Check the ``Create roles and permissions boundary`` option under Permissions and click on Create. This will take a few minutes for the application to create. An API endpoint is created which is backed by a Lambda function and a Dynamo DB. 
    ![awsserverlessapp](/images/awsserverlessapp.png)
    ![codetablambda](/images/codetablambda.png)

    The Architecture created is as below:
    ![serverlessArchitecture](/images/serverlessArchitecture.png)

9. Navigate to the github repository that is created in your github account. The repository is a SAM application. Please refer to the [SAM documentation](https://docs.aws.amazon.com/s.erverless-application-model/) for more information.
    Some key files to note here are:
    |Folder/File name|Purpose|
    |--------|-------|
    |template.yml| An AWS SAM template file closely follows the format of an AWS CloudFormation template file. The template file specifies the resources and the events thats will be created by the SAM CLI|
    |buildspec.yml| The buildspec.yml specifies the phases within the CodeBuild project |
    |src/handlers| This folder specifies the code for the lamda handlers|
    |event| The folder specifies the API endpoints that will be created by the SAM|

10. Navigate to the serverless [application](https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/applications) created and access the ``serverless-start`` application. Copy the API Endpoint for the serverless application.

11. Now we will use POSTMAN to create a few records using the API Endpoint as below:
    ![postmanPOST](/images/postmanPOST.png)
    Feel free to post additional rows.
    These operations will create Items in the DynamoDB ``serverless-start-SampleTable-xxxxxxx`` table.
    ![dynamoDBItems](/images/dynamoDBItems.png)

12. We will now get the list of items. Copy the API Endpoint in your browser console and hit enter. e.g. ``https://xxxxxxxxx.execute-api.ap-southeast-2.amazonaws.com/Prod/``. The list of items in the ``serverless-start-SampleTable-xxxxxxx`` table will be displayed as below:
    ![listallItems](/images/listallItems.png)

13. Next lets try to fetch an item that does not exist in the table. In the below screenshot we are fetching item with an id = 4. Currently when an item is not found in the backend a blank page is displayed.
    ![emptystringwhenpickingid](/images/emptystringwhenpickingid.png)

11. Next we will open an IDE to update the serverless application created, to return an error if an item requested does not exist. Instructions on how to use an IDE of choice can be found in Code tab under Developer Tools. 
    ![developertools](/images/developertools.png)
10. For this lab we will use the VS Code IDE. Follow [instructions](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/welcome.html) to configure your VS Code instance to work with the github repository created by the serverless application deployment. Authorize VS Code to work with your github account and checkout the github repository created by the lambda application.
11. Open ``get-by-id.js`` under ``src/handlers`` and replace

````
    const { Item } = await docClient.get(params).promise();

    const response = {
        statusCode: 200,
        body: JSON.stringify(Item),
    };
````
with 
```    
    const { Item } = await docClient.get(params).promise();
    let bodyString
    if (Item) bodyString = JSON.stringify(Item)
    else bodyString = JSON.stringify({"error": "Item not found" })

    const response = {
        statusCode: 200,
        body: bodyString,
    };
```
12. Once done commit these changes and this will kick off the pipeline created by the serverless app.
 ![pipelinestarted](/images/pipelinestarted.png)

13. Once deployed by the pipeline, navigate to browser fetch a particular id via the API endpoint.
``https://sxz1x9e5r8.execute-api.ap-southeast-2.amazonaws.com/Prod/1``
This will result in following output as updated in the source code.
 ![errorwhenidnotfound](/images/errorwhenidnotfound.png)

## Safe Deployments
We will use 
SAM a macro on top of cfn
SAM support use parameter, mappings and import.
Intrinsic functions also support.
ImportValue to 

AutoPublishAlias property

For this step we will perform a safe deployment. 
Using SAM you can take advantage of gradual deployments. 
Some safe depoloyments include:
- Canary Deployments
- Linear Deployments

### Create a few items via the API GW
1. Navigate to the API Gateway and filter for the API GW that starts with the text ``serverless-start``
2. Click on the POST method and then click on Test button on the POST-Method Execution pane
3. On the ``POST - Method Test`` screen enter the request body as below:
        ```
        {
            "id": "1",
            "name": "Canary"
        }
        ```
        You can enter a few more records as you please.
4. Once POSTed, when fetching a GET on a API gateway you should see the records as below.
    ![apigetrecords](/images/apigetrecords.png)

### Changing the error message when an item is not found
1. Navigate to VS Code and update the error message in the file ``src\handlers\get-by-id.js`` when an item id is not found in the DynamoDB. The new error message is ``Item not found in the catalog``
    ![updatederrormessage](/images/updatederrormessage.png)
2. Navigate to SAM ``template.yaml`` file and add instructions for a Canary Deployment.
3. Under the ```` resource within SAM template add the below lines below Runtime. 
```
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Canary10Percent5Minutes
```
4. Ensure that the indendation is accurate.
    ![updatedsamtemplate](/images/updatedsamtemplate.png)
5. Commit and push the changes to github. This will start the deployment pipeline.

