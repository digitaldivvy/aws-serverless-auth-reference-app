# Serverless Authentication and Authorization Quickstart Lab

This lab demonstrates how to demo the SpaceFinder reference app, and help you understand the interactions between Cognito User Pools, Cognito Federated Identities, API Gateway, Lambda, and IAM.

>**Estimated cost:** The estimated cost to run this lab for an hour is $0 for AWS Accounts eligible for the [AWS Free Tier](https://aws.amazon.com/free/), and less than $0.25 for other AWS accounts.

> Please be sure to complete the "CLEANUP" section after you're done with the lab!

# Here's the high-level plan...

1. **SETUP** (5 minutes): Provision an EC2 instance, and run the Docker container. Configure your AWS credentials, and deploy the AWS resources. Start Ionic 2 server.
1. **INTERACT AND LEARN** (30 minutes): Interact with the hybrid mobile app, and gain insights with the behind-the-scenes info displayed in the browser's JavaScript console. Explore how the AWS resources are configured.
1. **CLEANUP** (1 minute): Stop the Ionic 2 server, and un-deploy the AWS resources.


# SETUP (5 minutes)

Setup is quick and easy. You'll provision an EC2 instance, and run the Docker container. Once your AWS credentials and configured and the AWS resources are deployed, you'll start the Ionic 2 server.

1. **Launch the EC2 instance** in your AWS account, using a public AMI which contains a Docker image with a pre-configured environment:

	- **Public AMI**: `ami-bada16ac`
	- **Instance type**: m4.large
	- **Subnet/AZ:** Launch in any **public** VPC subnet (in any AZ)
	- **Public IP**: Enable "Auto-assign Public IP"
	- **Security Groups:** Open up ports 22 (SSH) and 80 (HTTP) to `0.0.0.0/0`
	- **SSH keypair:** Associate it with an SSH keypair of your choice
1. **SSH into the EC2 instance**.

		ssh -i /path/to/keypair.pem ec2-user@PUBLIC_DNS_OF_YOUR_EC2_SERVER

1. **Once logged in, run the Docker container** in interactive mode. This  command will start the Docker container, and bind the container’s port 8100 to the host EC2 instance’s port 80:

		docker run --rm -it -p 80:8100 awsdevops/aws-serverless-auth-reference-app

	> Troubleshooting: If you encounter a error saying “Bind for 0.0.0.0:80 failed: port is already allocated”, try: `sudo service docker restart`

1. **Create a highly-privileged IAM user, if necessary.** The lab needs permissions to provision/de-provision Cognito User Pools, DynamoDB tables, S3 buckets, Lambda functions, API Gateway configurations, CloudFormation stacks, and IAM roles.
	> All resources are created in your personal AWS account. This lab is self-contained and respectfully cleans up after itself by un-deploying all generated AWS resources.
1. **Configure the AWS credentials** to be used while inside the Docker container.

		aws configure

	> Just accept the defaults when it asks for region and output format. The app configuration file at `~/api/config.js` configures the resources to be provisioned in `us-east-1` by default. Also, AWS credentials are NOT persisted between Docker container runs, so if you exit and later re-run the Docker container, remember to run `aws configure` again.

1. **Deploy the AWS resources into your account**

		cd /home/aws-serverless-auth-reference-app/api
		gulp deploy
		gulp bootstrap

1. **Start the Ionic 2 server.** This starts up the Ionic 2 server listening on port 8100, which is port-mapped (via Docker) to port 80 of the host EC2 instance.

		cd /home/aws-serverless-auth-reference-app/app
		ionic serve
1. **View the hybrid mobile app in your browser**, by visiting:

		http://PUBLIC_DNS_OF_YOUR_EC2_SERVER/

1. **Open the JavaScript console**. As you interact with the hybrid mobile app, useful behind-the-scenes info will appear in the JavaScript console.

	* For example, for Firefox on Mac, the JavaScript console can be toggled via the menu: `Tools > Web Developer > Web Console`. For Chrome on Mac, the menu is: `View > Developer > JavaScript Console`.
	* You may find it helpful to have the JavaScript console display docked on the right side of the browser main window. This will allow you to see both mobile app as well as the JavaScript console output at the same time.

1. **Resize the browser main window**, to simulate the width of a mobile phone. Please be sure that you can see output displayed in the JavaScript console, as you click around the app.

# INTERACT AND LEARN (30 minutes)

Interact with the mobile app, and gain insights by viewing the behind-the-scenes info in the browser's JavaScript console. Explore how the AWS resources are configured.

---

### A. Sign-up and Sign-in

1. **Sign-up as new user in the hybrid mobile app, using your e-mail address**
1. **Provide confirmation code from e-mail to validate e-mail and confirm registration**
1. **Sign-in as your new user**
	- Review the output in the browser's JavaScript console.
      - *Which JWT tokens are returned from Cognito User Pools?*
      - *Which AWS credential components are returned from Cognito Federated Identities?*
	- Copy/paste the identity token into the JWT debugger at `http://jwt.io`, and decode it to see the base64-decoded content.
		- *How long is the identity token valid for before it expires?*
        - *Which attributes are encoded in the token?*
	- Copy/paste the access token into the JWT debugger at `http://jwt.io`, and decode it to see the base64-decoded content.
		- *How long is the access token valid for before it expires?*
        - *Which attributes are encoded in the token?*
        - *Which attributes also exist in the identity token? Which ones are different?*
1. **Copy the identity token displayed in the JavaScript console. This will be needed later.**

---

### B. API Gateway authorization using User Pools Authorizer

1. **Browse to "Resources" tab in the mobile app**
1. **Attempt to load locations "without Auth"**
    - *Based on the JavaScript console output, what was the URL path of the API request that was issued?*
    - *For this User Pools Authorizer request, which HTTP headers were sent?*
	- *What HTTP status code is returned?*
	- *Bonus question: Which HTTP headers are in the HTTP response? (Hint: Look at the Developer Tool's Network tab)*
1. **Attempt to load locations "with Auth"**
    - *What was the URL path of the API request that was issued?*
    - *For this User Pools Authorizer request, which HTTP headers were sent as part of the HTTP request?*

---
### C. API Gateway authorization using IAM Authorization
1.	**Click on a location**
	- *For this IAM Authorization request, which HTTP headers were sent?*
	- *How are the HTTP headers sent for IAM Authorization requests different than those sent for User Pool Authorizer requests?*
	- *Note that for IAM Authorization requests, a signature ("SigV4 signature") is calculated and sent as HTTP request headers. Note that User Pool Authorizer requests do not make use of any signatures.*
1.	**Select a resource (desk or conference room). Book one of the available timeslots.**
	- *Which HTTP method (GET/POST/PUT/DELETE) was called?*
	- *For this IAM Authorization request, which HTTP headers were sent?*
	- *Note that with IAM Authorization signing (aka "SigV4 signing"), the request URI, request body, and HTTP method are all used to calculate the signature.*
1. **Browse to "Bookings" tab, and cancel the booking you just made.**
	- *Which HTTP method (GET/POST/PUT/DELETE) was called?*

---

### D. S3 authorization using IAM Authorization

1. **Browse to "Account" tab.**
1. **Select to upload an local image as your profile picture. Any image will do.**
    - *This image will be stored directly in the S3 bucket that was created in your AWS account as part of the setup process. The image was uploaded directly to S3 using the AWS JavaScript S3 SDK (that is, it didn't go through API Gateway). The authorization headers are the same headers as those used earlier by API Gateway when configured to use IAM Authorization. This is because both API Gateway's IAM Authorization and the AWS SDKs rely on the same standard "SigV4 signing" method.*
	- *Temporary AWS credentials were used to sign the PUT request to the S3 API. How were these temporary AWS credentials obtained?*

---

### E. Fine-grained access control using IAM Authorization

1.	**You are currently logged in without Admin privileges. Toggle on "View Admin features", available in the "Account" tab"**. Note that you are NOT signed in currently as an administrator, so even though you can enable the display of Admin-only features, note that due to API authorization settings, the Admin-only API calls will still not work.
1.	**Browse back to "Resources" tab**
1.	**Click on "Add a location"**
1.	**Fill in requested information and click on the “Add location” button**
	- *As expected, your API request is rejected because you don't have permissions to perform this API backend operation (because your account is not in the administrators group).*
	- *Bonus question: The HTTP response body will explain why your API request was denied. What is the HTTP response body? (Hint: Look at the Developer Tool's Network tab)*

---

### F. Fine-grained access control using Custom Authorizers

1. **Click on the arrow located in the upper left of the app (next to the `Add a Location` title), to return to the previous screen.**
1. **Click on "Load locations with Auth", and click on a location of your choice.**
1. **When viewing the resources for the location, click on the "Delete" button next to one of the conference rooms/desks. Click "OK" to confirm when the delete confirmation dialog pops up.**
    - *An API Gateway Custom Authorizer is associated with this API operation. The Custom Authorizer logic evaluates the user's identity token and corresponding Cognito User Pools group memebership. Since your user isn't in the administrators group, the Custom Authorizer returns an IAM policy that effectively denies the user from performing this API operation.*

---
    
### G. Testing User Pool Authorizers using the API Gateway console

1.	**In a new browser tab/window, sign in to the AWS Management Console. Browse to the API Gateway console.**
1.	**Click to view the details for the "Spacefinder-API"**
2.	**In the lefthand menu, click "Authorizers" to view the Authorizers associated with the 'Spacerfinder-API'.**
3.  **View the details of the "spacefinder-userPool-authorizer"**
4.  **Copy your user's identity token from the JavaScript console in the other browser tab/window. (You'll need to scroll back to the beginning of your JavaScript console to copy the identity token.)**
5.  **Return back to the "spacefinder-userPool-authorizer" detail screen. In the righthand pane, scroll down to the "Test your authorizer" section. Paste in the identity token into the "Identity token" textfield, and click "Test".**
    - *Assuming you pasted a valid token, you should see your decoded identity token in the "Claims" textbox. This signifies that you would pass this yes/no authorization check.*
6. **Make some small, random edit to the value of the "Identity token" textfield, and click "Test" again.**
    - *You should now see an "Unauthorized request" message, which indicates that if this invalid token were to be sent to this API operation, it would     result in a denied request.*
    
---

### H. Testing Custom Authorizers using the API Gateway console
    
1.  **While still on the "Authorizers" screen, click on 'spacefinder-custom-authorizer' to view the details for that authorizer.**
1.  **In the righthand pane, scroll down to the "Test your authorizer" section. Paste in the identity token into the "Identity token" textfield, and click "Test"**
    - *You should see an IAM Policy with permissions to Allow restricted access to the API operations. The IAM Policy also has explicit Deny statements to block the creation or deletion of locations/resources.*

---

### I. Logging in as an Admin

1. **Sign out, and sign back in as an Admin. Administrators can create and delete locations/resources. Sign-in as username `admin1` and default password `Test123!`**
1. **Copy the identity token displayed in the JavaScript console. Be sure to get the most recently displayed identity token.**
1. **Browse to "Resources" tab.**
1. **Click on "Load locations with Auth", and click on a location of your choice.**
1. **When viewing the resources for the location, click on the "Delete" button next to one of the conference rooms/desks. Click 'OK' to confirm when the delete confirmation dialog pops up.**
    - *An API Gateway Custom Authorizer is associated with this API operation. The Custom Authorizer logic evaluates the user's identity token and corresponding Cognito User Pools group membership. Since your user is in the administrators group, the Custom Authorizer returns an IAM policy that effectively allows the user to perform this API operation.*
1. **Click on the arrow located in the upper left of the app (next to the `Resources` title), to return to the previous screen.**
1. **Click on "Add a location"**
1.	**Fill in requested information and click on the “Add location” button**
	- *This time, your API request is successful, since your account is part of the administrators group, which is associated with an IAM role that has permissions to perform this API operation.*

---

### J. Testing Authorizers using the API Gateway console, part 2

1.	**Return to the browser tab/window showing the API Gateway console.**
1.  **Navigate to the 'spacefinder-userPool-authorizer' detail screen. In the righthand pane, scroll down to the "Test your authorizer" section. Paste in the admin user's identity token into the "Identity token" textfield, and click "Test".**
    - *Assuming you pasted a valid token, you should see your decoded identity token in the "Claims" textbox. This signifies that you would pass this yes/no authorization check.*
1.  **While still on the "Authorizers" screen, click on 'spacefinder-custom-authorizer' to view the details for that authorizer.**
1.  **In the righthand pane, scroll down to the "Test your authorizer" section. Paste in the admin user's identity token into the "Identity token" textfield, and click "Test".**
    - *You should see an IAM Policy with permissions to Allow access to the API operations. Notice that the explicity Deny statements (when we tested earlier using the identity token associated with a non-Admin) are no longer present in the IAM Policy.*

---

### K. Exploring custom authorizer implementation details

1.	**Browse to the AWS Lambda console**
1.	**Click "Create a Lambda function"**
1.	**Enter "authorizer" in the filter search box**
1.	**Choose the custom authorizer Lambda blueprint for either Node.js or Python**
1.	**Do not add any triggers and click next**
1.	**Scroll down to the in-line text editor to see the Lambda blueprint sample code.**
1.	**Browse to <https://github.com/awslabs/aws-serverless-auth-reference-app/blob/master/api/lambda/authorizer.js>**
	- *This is the source code of the custom authorizer used for this project.*
	- *Which validations are performed against the HTTP request's authorization header passed to Lambda as part of the event object?*
	- *Since a custom authorizer should always return an effective IAM policy for a token sent to it, the project returns a “deny-all” policy for any invalid or malformed tokens.*
	


# CLEANUP (1 minute)

Cleanup is fast and easy. This lab is self-contained and cleans up after itself by un-deploying all auto-generated AWS resources).

1. **Exit Ionic 2 console and stop the Ionic server.**
	- `Control-C` or `q` (for quit) will exit the Ionic 2 console.

1. **Un-deploy all AWS resources**:

		cd /home/aws-serverless-auth-reference-app/api
	 	gulp undeploy

1. **Exit the Docker container**

	- Type `exit` at the command line.

1. **Terminate the EC2 instance.**