To construct the tutorial application and the Lambda function it uses

Create a working directory on your computer for this tutorial.

change diectory to your workdirectory using CD command 

      cd d:/yourworkdirectory

install the aws SDK 

      npm install aws-sdk
      
On Linux or Mac let's use ~/MyLambdaApp; on Windows let's use C:\MyLambdaApp. From now on we'll just call it MyLambdaApp.

Download slotassets.zip here

navigate to the link below and press download into your prefered directory.
      
      https://github.com/awsdocs/aws-doc-sdk-examples/blob/master/javascript/example_code/lambda/tutorial/slotassets.zip

This archive contains the browser assets that are used by the application, the Node.js code that's used in the Lambda function, and several setup scripts. In this tutorial, you modify the index.html file and upload all the browser asset files to an Amazon S3 bucket you provision for this application. As part of creating the Lambda function, you also modify the Node.js code in slotpull.js before uploading it to the Amazon S3 bucket.

Unzip the contents of slotassets.zip as the directory slotassets in MyLambdaApp. The slotassets directory should contain the 30 files.

Create a JSON file with your account credentials in your working directory. This file is used by the setup scripts to authenticate their AWS requests. For details, see Loading Credentials in Node.js from a JSON File

      https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/loading-node-credentials-json-file.html
  
Name the file Config.json 
Make sure the file content looks something like this

      {
        "accessKeyId": "AKIASLAAKDIFJRMKIUHS",
        "secretAccessKey": "d5xfqiseLOrMNiLaSDFrE0a3oIL+IVSNgAQK4dGg",
        "region": "us-east-2"
      }

making sure you use your own ACCESS_KEY_ID and SECRET_ACCESS_KEY instead and the region where you are testing the solution.

copy the Config.json file into the unzipped directory of slotassets extracted earlier.


      ****************************************************************************************************

For this application, the first thing you need to create is an Amazon S3 bucket to store all the browser assets. These include the HTML file, all graphics files, and the CSS file. The bucket is configured as a static website so that it also serves the application from the bucket's URL.

The slotassets directory contains the Node.js script s3-bucket-setup.js that creates the Amazon S3 bucket and sets the website configuration.

To create and configure the Amazon S3 bucket that the tutorial application uses

At the command line, type the following command, where BUCKET_NAME is the name for the bucket:

      node s3-bucket-setup.js BUCKET_NAME

The bucket name must be globally unique. If the command succeeds, the script displays the URL of the new bucket. 

      Make a note of this ARN because you'll use it later.

      ******************************************************************************************************
 
Prepare an Amazon Cognito Identity Pool
The JavaScript code in the browser script needs authentication to access AWS services. Within webpages, you typically use Amazon Cognito Identity to do this authentication. First, create an Amazon Cognito identity pool.

To create and prepare an Amazon Cognito identity pool for the browser script

      Open the Amazon Cognito console, 

      Create new identity pool.

      Enter a name for your identity pool, 

      choose enable access to unauthenticated identities, 

      and then choose Create Pool.

      Choose View Details to display details on both the authenticated and unauthenticated IAM roles created for this identity pool.

      In the summary for the unauthenticated role, c

      hoose View Policy Document to display the current role policy.

      Choose Edit to change the role policy, and then choose Ok.

In the text box, edit the policy to insert this "lambda:InvokeFunction" action, so the full policy becomes the following.

      {
         "Version": "2012-10-17",
         "Statement": [
         {
            "Effect": "Allow",
            "Action": [
               "lambda:InvokeFunction",
               "mobileanalytics:PutEvents",
               "cognito-sync:*"
            ],
            "Resource": [
               "*"
            ]
          }
        ]
      }
Choose Allow.

Choose Sample code in the side menu. Make a note of the identity pool ID, shown in red text in the console.

<img width="500" alt="identity-pool-id" src="https://user-images.githubusercontent.com/14894918/114202944-5f513600-9960-11eb-8a45-1d91d4bbbb92.png">


                    
Edit the Browser Script
Next, update the browser script to include the Amazon Cognito identity pool ID created for this application.

To prepare the browser script in the webpage

Open index.html in the Workdirectory folder in a text editor.

Find this line of code in the browser script.

      AWS.config.credentials = new AWS.CognitoIdentityCredentials({IdentityPoolId: 'IDENTITY_POOL_ID'});

Replace IDENTITY_POOL_ID with the identity pool ID you obtained previously.

Save index.html.

      ************************************************************************************************************
A Lambda function requires an execution role created in IAM that provides the function with the necessary permissions to run.

      Open lambda-role-setup.js in the slotassets directory in a text editor.

      Find this line of code.

      const ROLE = "ROLE"

      Replace ROLE with another name.

      Save your changes and close the file.

      At the command line, type the following.

      node lambda-role-setup.js

      Make a note of the ARN returned by the script. You need this value to create the Lambda function.


      ***********************************************************************************************************
  
The Lambda function generates three random numbers, then uses those numbers as keys to look up file names stored in an Amazon DynamoDB table. In the slotassets.zip archive file are two 

Node.js scripts named 

      ddb-table-create.js 
and 

      ddb-table-populate.js

Together these files create the DynamoDB table and populate it with the names of the image files in the Amazon S3 bucket. The Lambda function exclusively provides access to the table. Completing this portion of the application requires you to do these things:

Edit the Node.js code used to create the DynamoDB table.

Run the setup script that creates the DynamoDB table.

Run the setup script, which populates the DynamoDB table with data the application expects and needs.

To edit the Node.js script that creates the DynamoDB table for the tutorial application

      Open ddb-table-create.js in the slotassets directory in a text editor.

Find this line in the script.

      TableName: "TABLE_NAME"

Change TABLE_NAME to one you choose. Make a note of the table name.

      Save and close the file.

To run the Node.js setup script that creates the DynamoDB table

At the command line, type the following.

      node ddb-table-create.js
      
Once the DynamoDB table exists, you can populate it with the items and data the application needs. The slotassets directory contains t Node.js script ddb-table-populate.js that automates data population for the DynamoDB table you just created.

To run the Node.js setup script that populates the DynamoDB table with data

      Open ddb-table-populate.js in a text editor.

Find this line in the script.

      var myTable = 'TABLE_NAME';

Change TABLE_NAME to the name of the table you created previously.

      Save and close the file.

At the command line, type the following.

      node ddb-table-populate.js
      
      ***********************************************************************************************************
The Lambda function is invoked by the browser script every time the player of the game clicks and releases the handle on the side of the machine. There are 16 possible results that can appear in each slot position, chosen at random.

The Lambda function generates three random results, one for each slot wheel in the game. For each result, the Lambda function calls the Amazon DynamoDB table to retrieve the file name of the result graphic. Once all three results are determined and their matching graphic URLs are retrieved, the result information is returned to the browser script for showing the result.

The Node.js code needed for the Lambda function is in the slotassets directory. You must edit this code to use it in your Lambda function. Completing this portion of the application requires you to do these things:

--Edit the Node.js code used by the Lambda function
--Compress the Node.js code into a .zip archive file that you then upload to the Amazon S3 bucket used by the application.
--Edit the Node.js setup script that creates the Lambda function.
--Run the setup script, which creates the Lambda function from the .zip archive file in the Amazon S3 bucket.

To make the necessary edits in the Node.js code of the Lambda function

the Node.js for the application is the slotpull.js

      Open slotpull.js in the slotassets directory in a text editor.

Find this line of code in the browser script.

      TableName: = "TABLE_NAME";
      
Replace TABLE_NAME with your DynamoDB table name.

Save and close the file.


To prepare the Node.js code for creating the Lambda function

Compress slotpull.js into a .zip archive file for creating the Lambda function.

Upload slotpull.js.zip to the Amazon S3 bucket you created for this app. You can use the following CLI command, where BUCKET is the name of your Amazon S3 bucket:

      aws s3 cp slotpull.js.zip s3://BUCKET
      
Upload the website files - the HTML, PNG, and CSS files - to the bucket.

      ******************************************************************************************
      
Creating the Lambda Function
You can provide the Node.js code for the Lambda function is in a file compressed into a .zip archive file that you upload to an Amazon S3 bucket. The slotassets directory contains the Node.js script lambda-function-setup.js that you can modify and run to create the Lambda function.

To edit the Node.js setup script for creating the Lambda function

Open lambda-function-setup.js in the slotassets directory in a text editor.

Find this line in the script

         S3Bucket: 'BUCKET_NAME',
         
Replace BUCKET_NAME with the name of the Amazon S3 bucket that contains the .zip archive file of the Lambda function code.

Find this line in the script

         S3Key: 'ZIP_FILE_NAME',

Replace ZIP_FILE_NAME with the name of the .zip archive file of the Lambda function code in the Amazon S3 bucket.

Find this line in the script.

         Role: 'ROLE_ARN',

Replace ROLE_ARN with the ARN of the execution role you just created.

      Save and close the file.

To run the setup script and create the Lambda function from the .zip archive file in the Amazon S3 bucket

At the command line, type the following.

      node lambda-function-setup.js


      Open a web browser.

      Open the index.html in the Amazon S3 bucket that hosts the application.



Pull the liver and Good luck...



