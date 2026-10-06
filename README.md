# 🚀 Auto-Deploy Static Website via AWS CodePipeline

This guide walks you through building a simple CI/CD pipeline using GitHub and AWS CodePipeline to automatically deploy a static HTML site to an S3 bucket.


## ✅ STEP 1: Create a GitHub Repo and Add `index.html`

### 🔧 What to do:
1. Go to [https://github.com](https://github.com)
2. Click **"New repository"**
   - **Repo name**: `my-static-site`
   - **Description**: “Simple HTML site for AWS CI/CD”
   - Public or private (your choice)
   - ⚠️ **DO NOT** check "Initialize with README" if you’re uploading manually
3. Once the repo is created, add a file named `index.html` with this content:

```html
<!-- index.html -->
<html>
  <head><title>My Deployed Site</title></head>
  <body><h1>Hello from GitHub + AWS CodePipeline!</h1></body>
</html>
```

## ✅ STEP 2: Create an S3 Bucket for Static Website Hosting
### 🔧 What to do:
1. Go to the [AWS S3 Console]((https://s3.console.aws.amazon.com/)
2. Click **"Create bucket"**
    - **Bucket name**: ``mycicdbucket7`` (must be globally unique!)
    - **Region**: Choose your preferred AWS region (e.g., ``us-east-1``)
    - Leave other settings as default
    - Scroll down to **Block Public Access settings** for this bucket. Uncheck “Block all public access”
    - Check the box acknowledging the security warning that the bucket contents will become public.
    - Click **Create bucket**.
   
## 🌐 Configure Static Website Hosting & Permissions
3. Click your newly created bucket name from the S3 list.
    - Open the **Properties** tab, scroll completely to the bottom to **Static website hosting**, and click Edit.
    - Select **Enable**.
    - Set **Index document**: ``index.html``.
    - Click **Save changes**.
    - Open the **Permissions** tab and scroll down to **Bucket policy** → click **Edit**.
    - Paste this exact configuration block *(replace ``mycicdbucket7`` with your exact bucket name if it's different)*:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-ci-cd-bucket/*"
    }
  ]
}
```
   - Click **Save changes**.

## ✅ STEP 3: Set Up CodePipeline (This Is the Main Dish)
### 🔧 What to do:
1. Go to the **AWS CodePipeline Console**.
2. Click **Create pipeline**.
3. Select **"Build custom pipeline"**

**A. Pipeline Settings**
 - **Pipeline name**: ``MyWebAppPipeline``
 - **Pipeline type**: Leave it at the modern default (**V2**).
 - **Execution mode**: **Queued** (Default)
 - **Service role**: Choose **New service role**.
 - Click **Next**.
    
**B. Source Stage**:
- **Source provider**: Select **GitHub (Version 2)**.
- **Connection**: Click **Connect to GitHub**.
   - Choose **GitHub via GitHub App** if prompted.
   - Click **Connect to GitHub** in the popup window, sign in, authorize **AWS CodeStar** to view your repositories, and follow the prompts to complete the installation link.
- **Repository name**: Select your ``my-static-site`` repository.
- **Branch name**: Select ``main``.
- **Output artifact format**: Leave as **CodePipeline default**.
- **Trigger configuration**: Leave as **No filter** (keeps instant push deployment running).
- Click **Next**.
    
**C. Build Stage**
- Click **Skip build stage** and confirm the prompt.
    
**D. Deploy Stage**
- **Deploy provider**: Select **Amazon S3**
- **Region**: Select the same region you used for your bucket (e.g., ``us-east-1``)
- **Bucket**: Select (```mycicdbucket7```)
- **CRITICAL CONFIGURATION**: Check the box for **Extract file before deploy**.
- Expand **Additional configuration** at the bottom of the section.
- **LATEST ACCESSIBILITY UPDATE**: Look for the **Canned ACL** dropdown menu field. Select ``Please select an option`` (leave it completely blank/unselected). Do **NOT** choose ``public-read``. This prevents the ``AccessControlListNotSupported error`` completely since your Bucket Policy from Step 2 manages internet visibility safely.
- Click **Next**, review your details, and click **Create pipeline**.


🎉 AWS will now start the pipeline immediately and attempt the first deployment!

## 🔥 STEP 4: Test the Pipeline
Make a small change in your index.html file to confirm the automation works.

**Example:**
Change this:
```html
<h1>Hello from GitHub + AWS CodePipeline!</h1>
```
To this:
```html
<h1>This site auto-deploys from GitHub to AWS S3. Cool, right?</h1>
```
Then:
1. Commit and push the updates directly into the ``main`` branch.
2. CodePipeline will auto-detect the change.
3. Open the AWS CodePipeline interface—your **Source** and **Deploy** steps will turn green sequentially.
4. Access your permanent global website address using the **Bucket URL**.
