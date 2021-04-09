To construct the tutorial application and the Lambda function it uses

Create a working directory on your computer for this tutorial.

install the aws SDK 

      npm install aws-sdk
      
On Linux or Mac let's use ~/MyLambdaApp; on Windows let's use C:\MyLambdaApp. From now on we'll just call it MyLambdaApp.

Download slotassets.zip here

navigate tp the link below and press download into your prefered directory.
      
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

      Make a note of this URL because you'll use it later.

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

<img width="300" alt="identity-pool-id" src="https://user-images.githubusercontent.com/14894918/114202944-5f513600-9960-11eb-8a45-1d91d4bbbb92.png">


                    
Edit the Browser Script
Next, update the browser script to include the Amazon Cognito identity pool ID created for this application.

To prepare the browser script in the webpage

Open index.html in the MyLambdaApp folder in a text editor.

Find this line of code in the browser script.

AWS.config.credentials = new AWS.CognitoIdentityCredentials({IdentityPoolId: 'IDENTITY_POOL_ID'});

Replace IDENTITY_POOL_ID with the identity pool ID you obtained previously.

Save index.html.
