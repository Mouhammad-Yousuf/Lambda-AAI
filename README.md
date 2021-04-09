To construct the tutorial application and the Lambda function it uses

Create a working directory on your computer for this tutorial.

On Linux or Mac let's use ~/MyLambdaApp; on Windows let's use C:\MyLambdaApp. From now on we'll just call it MyLambdaApp.

Download slotassets.zip from the https://github.com/awsdocs/aws-doc-sdk-examples/blob/master/javascript/example_code/lambda/tutorial/slotassets.zip. This archive contains the browser assets that are used by the application, the Node.js code that's used in the Lambda function, and several setup scripts. In this tutorial, you modify the index.html file and upload all the browser asset files to an Amazon S3 bucket you provision for this application. As part of creating the Lambda function, you also modify the Node.js code in slotpull.js before uploading it to the Amazon S3 bucket.

Unzip the contents of slotassets.zip as the directory slotassets in MyLambdaApp. The slotassets directory should contain the 30 files.

Create a JSON file with your account credentials in your working directory. This file is used by the setup scripts to authenticate their AWS requests. For details, see Loading Credentials in Node.js from a JSON File.
