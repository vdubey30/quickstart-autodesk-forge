

0.Fork the following GitHub repository in your account: [https://github.com/vsnyc/quickstart-autodesk-forge/](https://github.com/vsnyc/quickstart-autodesk-forge/)

1.Go to AWS Console and choose the N. California Region.
	
1.1.Go to EC2 console and let's create a key pair named: `forge-demo`. It will download the forge-demo private key file to your computer

`NOTE This N. California region will be use for our Build Tests`

2.Go to AWS Console and choose the Oregon Region.

2.1.Go to EC2 console and let's create a key pair named: `forge-demo`. It will download the forge-demo private key file to your computer

`NOTE We will continue working on the Oregon Region.`

3.Go to Cloud9, create a new environment with instance type: t3.medium (see [this](http://workshop.quickstart.awspartner.com/10_prerequisites/20_workspace.html) for details)

4.Create [GitHub token](http://workshop.quickstart.awspartner.com/10_prerequisites/30_github_token.html).

5.Configure git in Cloud9 IDE, substituting your GitHub user name and email inside the quotes in the following commands:

```
git config --global user.name "YOUR_GITHUB_USERNAME" 
git config --global user.email "YOUR_GITHUB_EMAIL"
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=3600'
```

6.Clone git repo in Cloud9 IDE, replacing `YOUR_GITHUB_USERNAME` with your GitHub user name:  

```
git clone --recursive https://github.com/YOUR_GITHUB_USERNAME/quickstart-autodesk-forge.git
```

7.Download workshop assets. This will be located at the Root level of the project together with the quickstart-autodesk-forge folder.

`curl -O https://cftests.s3.amazonaws.com/forge-workshop/forge-workshop-assets.zip`

8.Unzip workshop assets  

`unzip forge-workshop-assets.zip`

9.Let's run the following command to make buckets and let's make a note of the created buckets. We will need the values of the `config bucket` and the `hosting bucket`  

`bash make_buckets.sh`

Sample output (you will get different values for the ID's):

```
make_bucket: au-demo-config-ab77e26d-758c
Created config bucket: au-demo-config-ab77e26d-758c
make_bucket: au-demo-code-ab77e26d-758c
Created code hosting bucket: au-demo-code-ab77e26d-758c
```

10.export the config bucket to an envioromental variable for convenience  

`export CONFIG_BUCKET=au-demo-config-ab77e26d-758c`

11.Open update_params.sh and fill lines 1-5. For line 6, you can either update it with your IP address (check at http://checkip.amazonaws.com/, e.g. "1.2.3.4\/32") or with "0.0.0.0\/0" to allow access from anywhere. Note the backslash for IP address value, it is required to escape it during substitution.

12.Now let's Execute the following command to use the update_params.sh new input values.

`bash update_params.sh`

13.Let's verify the substituted tokens in `forge-prod-us-west-2.json`, `taskcat_project_override.json`

14.Make a zip file to upload to config bucket  

`bash make_zip.sh`

15.Let's Upload the `quickstart-autodesk-forge.zip` and the test config file  `taskcat_project_override.json` to config bucket  

```
aws s3 cp quickstart-autodesk-forge.zip s3://$CONFIG_BUCKET/
aws s3 cp taskcat_project_override.json s3://$CONFIG_BUCKET/
```
16.In a new tab, open the following [launch stack link](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=Forge-App-CICD&templateURL=https://cftests.s3.amazonaws.com/quickstart-taskcat-ci/templates/taskcat-cicd-pipeline.template.yaml&param_ProdStackName=Forge-Prod-Stack&param_ProdStackConfig=forge-prod-us-west-2.json&param_TemplateFileName=autodesk-forge-master.json&param_TestStackConfig=taskcat_project_override.json&param_SourceRepoBranch=develop&param_ReleaseBranch=master&param_QSS3KeyPrefix=quickstart-taskcat-ci/&param_QSS3BucketName=cftests&param_GitHubRepoName=quickstart-autodesk-forge) that will setup your CodePipeline. Most fields are populated with defaults, fill in only the blank fields.

* Repository owner: your GitHub user name
* OAuth2 token: your GitHub oauth token created in Step 4
* Email: your email address
* ConfigBucket: your config bucket created in Step 9
* CodeHostingBucket: your code hosting bucket created in Step 9

17.After the `Forge-App-CICD` stack is created, it will automatically execute the CodePipeline. It takes about 25 minutes for the pipeline to reach the PROD stage. 

Go to the pipeline service and manually approve the last step in the PROD Stage. 

After that, you should see `Forge-Prod-Stack` being created in your account. It takes around 18 minutes for the `Forge-Prod-Stack` to complete. After that, go to the CloudFormation outputs and test your application going to the link provided by the value of `ForgeAppURL`.

18.Making changes to the app. We'll now replace the default app with a app that contains the dashboard. In the Cloud9 IDE:
    
   1. In terminal, go to the repo directory: `cd quickstart-autodesk-forge`
   2. Checkout develop branch: `git checkout develop`
   3. open `quickstart-autodesk-forge/templates/autodesk-forge-nodejs.json`. Change `FORGE_APP_NAME` in line 147 to `forge-viewmodels-nodejs-aws-dashboard`. Save the file.
   4. open `quickstart-autodesk-forge/templates/autodesk-forge.json`. Change `Toggle` value in line 801 from "false" to "true". Save the file.
   5. In terminal, from the quickstart-autodesk-forge directory, add the files, commit and push.
        
      ```
        git add -A
        git commit -m "Updated app to include a dashboard"
        git push
      ```

19.When new code is pushed to git, the pipeline invocation should start again. After the tests pass, you will have to approve the last stage for your Forge-Prod-Stack to be updated.

20.Once update is complete, go to the `ForgeAppURL` and verify that your application now shows a dashboard for your models. (Tested in Chrome browser)



