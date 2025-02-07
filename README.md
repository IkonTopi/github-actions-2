layout: post title: GitHub Actions - Creating a continuous deployment pipeline from GitHub to S3 date: 2025-02-03 17:32 +0000 tags: github actions cicd cd

toc: levels: [1, 2, 3, 4] categories:

- Ohjelmistokehityksen työvälineet
This page contains information on how to utilise GitHub Actions to build a Continuous Delivery pipeline setup from a github repository and it's main branch to an S3 Bucket with Static Website hosting enabled. The example is fairly simple and doesn't have a build process included. Remember to use us-east-1 region for the assignment.

AWS - S3 Static Website Hosting and Automation with GitHub Actions
{: .prompt-info }

Remember to check your AWS CLI Credentials from the AWS Academy Learner Lab environment. You need these for Git Hub Actions. These include the access key, secret key and session token!

AWS Credential information - Show CLI

AWS Credential information - Access key, Secret Key and Session Token parameters

1. Download and Prepare the Static Theme
Go to Free-css and search for the Browny theme.
Download the theme to your computer.
Extract the downloaded .zip file to a directory such as ~/LearningGit.
Navigate to the directory and initialize a git repository:
cd ~/LearningGit/Browny
git init
5. Modify the contents of your theme to include a nice little profile of you, and update some text to the index.html file.

2. Create an AWS S3 Bucket and Enable Static Website Hosting
Log in to AWS Academy Learner Lab.
Go to the S3 service and create a new S3 bucket. You can find your bucket on the "general purpose buckets" view once you've created it.
Set a unique name for your bucket, e.g., my-static-website-12345.
Disable the Block all public access setting, acknowledge that content will be public and visible to everyone.
Define a bucket policy to allow public access. Be sure to replace the BUCKET_NAME parameter with the unique name of your bucket. You can see your bucket name when you edit the bucket policy from the Permissions subpage :
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/*"
    }
  ]
}
Go to the Permissions tab and add the following JSON code under Bucket Policy (replace BUCKET_NAME with your bucket's name):
Enable Static Website Hosting: * Go to the Properties tab. * Find the Static Website Hosting section and enable it. * Set index.html as the start page. * Copy the provided bucket website endpoint (URL) to your web browser and check that the site loads properly.

3. Link Local Git Repository to GitHub
Create a private GitHub repository.
Follow GitHub’s instructions to link your local repository to GitHub. Be sure to change your username and the name of your repository:
git remote add origin https://github.com/USERNAME/GITHUB_REPO.git
git branch -M main
git push -u origin main
4. Automation: GitHub Actions Workflow
Create a .github/workflows/deploy.yml file in your repository and add the following content, be sure to replace BUCKET_NAME with the name of your S3 bucket. You can get the access key id, secret key and session token from the AWS Academy Learner Lab -> Details -> Show AWS CLI details button. Remember that these are valid 4 hours, and as long as your lab environment is running!
Copy the content below to the deploy.yml file:

name: Deploy to S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with: {% raw %}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
          aws-region: us-east-1  # Change region if needed
        {% endraw %}
      - name: Deploy files to S3
        run: |
          aws s3 sync . s3://BUCKET_NAME --delete
Add AWS credentials to GitHub Secrets, and add new Repository Secret parameters (Settings > Security -> Secrets and variables -> Actions):
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
These parameters should match the AWS Academy Learner Lab environment and details of your lab (see the beginning of instructions).

5. Test Automatic Deployment
Make local changes to index.html:
Update your name, image, and student ID.
Save changes and push them to GitHub:
git add .
git commit -m "Updated theme content"
git push
Check the GitHub Actions tab to ensure the workflow runs successfully.
Open your S3 bucket website URL to verify that the updated website is displayed correctly.
This guide covers static website hosting on AWS S3 and GitHub Actions automation. 🌍🚀
