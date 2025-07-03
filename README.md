# 🚀 Deploy a Static Website from GitHub to S3 using AWS CodePipeline

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
1. Go to the S3 Console
2. Click **"Create bucket"**
    - Bucket name: mycicdbucket7 (must be globally unique!)
    - Region: Choose your preferred AWS region (e.g., us-east-1)
    - Leave other settings as default
    - Uncheck **"Block all public access"**
    - Confirm the warning that shows up
3. After the bucket is created:
    - Click your bucket name
    - Go to the **Properties** tab
    - Scroll to Static website hosting
      - Enable **static website hosting**
      - Set:
          - **Index document**: ```index.html```
          - **Error document**: (optional) ```error.html```
      - Save changes
    - Now go to the **Permissions** tab
      - Set a bucket policy that allows public read access
        🛡️ Sample S3 Bucket Policy
        - Replace ```my-ci-cd-bucket``` with your actual bucket name:
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

## ✅ STEP 3: Set Up CodePipeline (This Is the Main Dish)
### 🔧 What to do:
1. Go to the AWS CodePipeline Console
2. Click **"Create pipeline"**
3. Select **"Build custom pipeline"**
4. Fill in the following sections:

📝 Pipeline Configuration
**A. Pipeline Settings**
 - **Pipeline name**: MyWebAppPipeline
 - **Service role**: Select “New service role”
 - Leave advanced settings as default
 - Click **Next**
    
**B. Source Stage**:
- **Source provider**: GitHub (Version 2) (GitHub v2 uses OAuth and is better supported)
- Click **"Connect to GitHub"** and authorize AWS to access your repos
- Choose your repo: ```my-static-site```
- Choose branch: ```main```
- **Change detection**: GitHub webhooks (auto-trigger on commit)
- Click **Next**
    
**C. Build Stage**
This is optional for static sites — we're skipping it for now.
- Choose: **Skip build stage**
- Confirm skip
- Click **Next**
    
**D. Deploy Stage**
- Deploy provider: **Amazon S3**
- Region: (same region as your bucket)
- Bucket: Select the bucket you created earlier (```mycicdbucket7```)
- Extract file before deploy: **YES**
Why? Because the pipeline will zip your site files during transfer. This option ensures S3 unpacks the zip and shows index.html directly.
- Click **Next**, then **Create Pipeline**

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
1. Commit and push the change to your **GitHub repo**
2. CodePipeline will auto-detect the change
3. The Source → **Deploy** stages will run automatically
4. Your updated site will appear live in your S3 bucket

🔗 **Access Your Live Website**
Use this URL format:
```text
http://<your-bucket-name>.s3-website-<your-region>.amazonaws.com
```
Example:
```text
http://my-ci-cd-bucket.s3-website-us-east-1.amazonaws.com
```
