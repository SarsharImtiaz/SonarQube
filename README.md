devsecops-pipeline-template
Adding the Sec in DevSecOps.

This is a template project to use when you need to add scanning and/or test automation into your existing GitLab runner pipelines using OpenShift.

Overview
This project implements a CI/CD pipeline using Tekton, integrated with GitLab and OpenShift. The pipeline is designed for the CCAM-API and includes code scanning with Fortify, publishing build results to Artifactory, and emailing scan results.

Key Steps of the Pipeline:
Checkout the sample-eight source code
Scan the source code with Fortify
Publish the build results to Artifactory
Email the scan results
Pipeline Parameters:
gitRepoName: the project name
gitPathName:
gitRevision: the git revision to check out and build
gitJobTime: the timestamp of the most recent commit
gitUserName: the user name of the user pushing the commits
fortifyScanType: can be either "quick" or "full"
emailRecipientList: list of email addresses (separated by commas) to receive the code scan results
Getting Started
Follow these steps to set up and run the CI/CD pipeline.

Prerequisites
Ensure the Tekton Catalog is installed into the namespace as the DevSecOps Tekton pipeline uses catalog tasks.
Setup GitLab Token
The pipeline needs credentials to interact with Artifactory and GitLab. Creating deploy tokens in GitLab provides controlled access to repositories without using personal credentials.

Log in to GitLab: Use your Jenne credentials.
Navigate to your project: Click on the project name from the GitLab dashboard or use the search bar.
Go to Project Settings: Click on Settings from the left sidebar menu.
Access Repository Settings: Click on Repository under the General section.
Deploy Tokens Section: Scroll down to the Deploy Tokens section and expand it.
Create a New Deploy Token: Click on the Create deploy token button.
Fill in Token Details:
Name: Descriptive name, e.g., tekton-<projectname>-deploy-token
Username: gitlab+deploy-token
Expires: Never
Scopes: read_repository, read_package_registry
Copy the Token: Copy the token immediately as it will not be shown again.
Update the token in the DevSecOps repository:

bash
Copy code
# Clone the repository
git clone git@cfsr.sso.dcn:aso/devsecops-pipeline-template.git

# Change directory to the cloned repository
cd devsecops-pipeline-template

# Update the secret-git-basic-auth.yaml file with the token
# Also update the creds-git.yaml file, using creds-git.yaml.template for the format

# Run the helm command to update OpenShift with the token
helm upgrade --namespace devsecops-pipeline-template --install -f tekton-pipeline/creds-git.yaml -f tekton-pipeline/creds-artifactory.yaml -f tekton-pipeline/creds-fortify.yaml devsecops ./tekton-pipeline
Setting Up GitLab Runner
A GitLab runner needs to be set up. Contact Jonathan D'Andries for assistance. Once set up, verify the configuration in GitLab under CI/CD settings.

Tags to be added in the .gitlab-ci.yaml file:

yaml
Copy code
  tags:
    - sampleeightball
    - aso
Triggering the Pipeline for Testing
The pipeline can be triggered manually using the curl command:

bash
Copy code
# Change the gitRevision, gitUserName, and emailRecipientList as needed
curl -i -X POST http://tekton-pipeline-devsecops.apps.dev.cloudapps-e.nsapps.dcn --header 'Content-Type: application/json' --data '{"gitRevision": "'"${CI_COMMIT_SHA}"'", "gitRepoName": "'"${CI_PROJECT_NAME}"'", "gitPathName": "'"${CI_PROJECT_NAME}"'", "userid" : "'"${AUTO_USERID}"'", "emailRecipientList" : "'"${SCAN_EMAIL_RECIPIENT_LIST}"'" , "gitJobID" : "'"${CI_JOB_ID}"'" , "manualAutoFlag" : "auto", "gitUserName" : "'"${GITLAB_USER_NAME}"'", "gitJobTime" : "'"$(TZ='America/New_York' date --date=${CI_JOB_STARTED_AT} --iso-8601=seconds)"'", "fortifyScanType": "'"$FORTIFY_SCAN_TYPE"'", "fortifyUploadFlag": "'"$FORTIFY_UPLOAD_FLAG"'"}'
Manual Pipeline Trigger Instructions
To manually trigger the pipeline:

Navigate to the Repository: Open the repository in your web browser.
Access the Pipeline Section: Look for the "Actions" or "Pipelines" tab.
Select the Pipeline to Trigger: Identify the pipeline associated with the branch or commit you want to trigger.
Initiate the Pipeline Manually: Click on the "Run pipeline" option.
Review Fortify Reports
HP Fortify performs static code analysis and provides recommendations for secure coding standards.

Run Fortify Scans: After the CI/CD pipeline completes, locate the Fortify scan results in Artifactory under aso-generic/xxxx/xxxxx.
Review Fortify Reports: Use Fortify Audit Workbench or the web interface to analyze the findings.
For more details, refer to the Fortify Audit Workbench Guide.

Contributing
Fork the repository.
Create a feature branch: git checkout -b my-new-feature
Commit your changes: git commit -am 'Add some feature'
Push to the branch: git push origin my-new-feature
Submit a pull request.
