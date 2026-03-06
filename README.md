# 🖼️ Serverless Image Resizer — AWS

A fully serverless image resizing pipeline built on AWS that automatically resizes images when uploaded to S3, reducing file size by up to **96%** with zero server management.

---

## 🏗️ Architecture

```
User Uploads Image (.jpg)
        ↓
S3 Input Bucket (my-image-input-2026)
        ↓  [S3 Event Trigger]
AWS Lambda Function (Python 3.11 + Pillow)
        ↓  [Resizes to 300x300]
S3 Output Bucket (my-image-output-2026)
        ↓
Resized Image Ready! ✅
```

---

## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon S3** | Store input and output images |
| **AWS Lambda** | Serverless function to resize images |
| **IAM Role** | Permissions for Lambda to access S3 |
| **Amazon CloudWatch** | Monitor logs and debug errors |
| **Pillow (Lambda Layer)** | Python library for image processing |

---

## ✅ Results

| | Details |
|---|---|
| Original Image Size | ~260 KB |
| Resized Image Size | ~12.9 KB |
| Size Reduction | **96%** |
| Resize Dimensions | 300 x 300 px |
| Trigger | Automatic on S3 upload |

---

## 📸 Screenshots

### 1. S3 Input Bucket — Image Uploaded
![Input Bucket](screenshots/input-bucket.png)

### 2. S3 Output Bucket — Resized Image Saved
![Output Bucket](screenshots/output-bucket.png)

### 3. CloudWatch Logs — Lambda Execution
![CloudWatch Logs](screenshots/cloudwatch-logs.png)

---

## 🧠 Lambda Function Code

```python
import boto3
from PIL import Image
import io

s3 = boto3.client('s3')
OUTPUT_BUCKET = 'my-image-output-2026'
SIZE = (300, 300)

def lambda_handler(event, context):
    input_bucket = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']
    
    print(f"Resizing image: {file_key}")
    
    response = s3.get_object(Bucket=input_bucket, Key=file_key)
    image_data = response['Body'].read()
    
    image = Image.open(io.BytesIO(image_data))
    image = image.resize(SIZE)
    
    buffer = io.BytesIO()
    image.save(buffer, format=image.format or 'JPEG')
    buffer.seek(0)
    
    output_key = f"resized-{file_key}"
    s3.put_object(Bucket=OUTPUT_BUCKET, Key=output_key, Body=buffer)
    
    print(f"Done! Saved to {OUTPUT_BUCKET}/{output_key}")
    return {"status": "success"}
```

---

## 🚀 How to Deploy

### Step 1: Create S3 Buckets
- Create `my-image-input-2026` (input bucket)
- Create `my-image-output-2026` (output bucket)
- Region: `us-east-1`

### Step 2: Create IAM Role
- Service: Lambda
- Policies: `AmazonS3FullAccess` + `AWSLambdaBasicExecutionRole`
- Role name: `lambda-image-resizer-role`

### Step 3: Create Lambda Function
- Runtime: Python 3.11
- Attach IAM role above
- Paste the code above
- Set timeout to **30 seconds**, memory to **512 MB**

### Step 4: Add Pillow Layer
```
arn:aws:lambda:us-east-1:770693421928:layer:Klayers-p311-Pillow:9
```

### Step 5: Add S3 Trigger
- Bucket: `my-image-input-2026`
- Event type: `PUT`
- Suffix: `.jpg`

### Step 6: Test
- Upload any `.jpg` to input bucket
- Check output bucket for `resized-` file ✅

---

## 🐛 Debugging

If image doesn't appear in output bucket:
1. Go to **CloudWatch → Log Groups**
2. Find `/aws/lambda/imageResizerFunction`
3. Check for timeout or permission errors

Common fixes:
- **Timeout error** → Increase Lambda timeout to 30s
- **Access Denied** → Check IAM role has S3 access
- **PIL not found** → Re-add the Pillow Lambda Layer

---

## 💡 Use Cases

- Social media apps (profile picture resizing)
- E-commerce (product image thumbnails)
- Portfolio websites (optimize performance)
- Any app needing automatic image compression

---

## 👨‍💻 Built With

- AWS Lambda (Serverless)
- Amazon S3
- Python 3.11
- Pillow Library
- Amazon CloudWatch

---

> **Note:** This project runs entirely within the AWS Free Tier limits.
