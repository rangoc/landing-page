# landing-page
Serverless landing page with static web hosting on S3 ( IaC ) ( S3 + ApiGateway + Lambda + DynamoDB + SES  )

# About the project
Landing page for your product, where you allow your potential customers to subscribe with their emails so you can notify them about the changes regarding the product, such as launching and details about development.
Whole project is based on Amazon's Web Services and whole infrastructure is written as a Cloudformation template, so to get it fully working only small modifications in a code will be needed, but it will be explained.

# How to set it up: 
Assuming that you already have AWS account set up, first thing you will need to do after cloning this project into your repository on a local machine is ```npm install```. 
If you already don't have serverless framework installed, this will be a good time to install it, so in your terminal go ahead with a command ```npm install -g serverless```.
Now, you are ready to deploy your project on a cloud, with a command: ```serverless deploy```.
After your stack has been created, you basically got your whole infrastructure made on a cloud, and only small changes have to be done from here onwards:

1. You will need to open your ApiGateway stages trough AWS Console and copy two urls. First url from **/lambdaWrite** resource's **POST** method, and second one from **/subscribe** resource's **POST** method.
  - First link you will need to paste in your **index.html** file in "**url**" field ( comment was added in index.html to help you with finding the specified field )
  - Second link you will need to paste in your **subscription.html** file in "**url**" field ( comment was added in subscription.html to help you with finding the specified field )

2. Now, after you've done that, you will need to **upload both index.html and subscription.html** files in a bucket that will be named something like "landing-pagebucket..." which will be **public** ( this is different from landing page serverlessdeploymentbucket, so make sure that you upload files to a right bucket)

3. After you uploaded both of those files in your bucket, you will need to **copy subscription.html file's Object URL** (which you can see if you click on the file inside of bucket) and **paste it inside of your stream-lambda.yml resource** in variable **subscriptionLink** ( comment was added to help you with finding it).
In that same file ( stream-lambda.yml ) make sure to add **Source** email/domain from which you want to send your emails. If you are in a sandbox mode, and you probably will be if you are using this project for testing purpose, make sure to first verify email adresses/domains in SES and then use them as Source and subscriberEmail.

4. Finally, do a ```serverless deploy``` command in your terminal, for all changes to be updated on your cloud and navigate back to index.html in your S3 bucket to test it out.

# How it works:
index.html in your S3 bucket will serve as main page of your static landing page web site.</br></br> User arrives on that page, enters his email and clicks "Subscribe". Under the hood, that email is being sent to ApiGateway which then triggers first lambda. That lambda writes that email to a DynamoDb table and uses that email as a primary key. Along with email, two attributes are added ( verified and verify_id ). Verified attribute represents the status of verification and is false by default. Verify_id is a randomly generated uuid.</br></br>
DynamoDb table has a stream enabled which second lambda listens to, filters all records on "insert" event and once triggered it will take verify_id from the inputted email and  that id will be used in a subscription link that's being sent to a user who has subscribed in order to uniquely identify him/her in a datebase and change his verified status to "true" later on.</br></br>
Verification email is sent to user's email box, once the user clicks it the link redirects him/her to subscription.html url which automatically triggers third lambda ( trough ApiGateway verify_id is being passed as accessToken) which will update the corresponding user's verified status to "true".</br></br>That's the whole architecture flow, easy peasy.

