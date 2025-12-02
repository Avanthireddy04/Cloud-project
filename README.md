# Cloud-project
## 📄 Culling Resumes for jobs using NLP Techniques

### Table of contents:

- [Project Overview](#-Project-Overview)
- [Architecture](#-Architecture)
- [Architecture Diagram](#-Architecture-Diagram)
- [Data Flow](#-Data-Flow)
- [Technology Stack](#-Technology-Stack)
- [Security](#-Security)
- [Limitations](#-Limitations)
- [Screenshots](#-Screenshots)



### 🎯 Project Overview
This Project Culling of Resumes using NLP Techniques hosted on AWS in simple is,(Resume Matching System) it is a cloud-based application deployed on AWS that automatically analyzes and ranks resumes against job descriptions using NLP and machine learning.
 It is designed to help recruiters automatically identify which candidates best match a given job description. Instead of manually reading through each resume, the system uses Natural Language Processing (NLP) and Machine Learning to read, analyze, and compare resumes with job requirements.

The process starts when a user uploads resumes and job descriptions through a web interface. These files are stored in Amazon S3. Whenever a document is uploaded, AWS Lambda automatically processes it by extracting the text, cleaning it, and converting it into a structured format. The cleaned data is then stored in Amazon DynamoDB.

The matching engine is hosted in a Streamlit application running on Amazon ECS Fargate. This engine takes the processed resume and job description text and converts them into vector embeddings using TF-IDF and BERT, which help the system understand both keyword importance and the deeper meaning behind the text. The system then calculates a similarity score for each resume using cosine similarity. The higher the score, the better the resume matches the job description.

Finally, the system displays the ranked results back to the user through the Streamlit interface. Recruiters can see which candidates best fit the job, along with details extracted from each resume. This automation reduces manual effort, improves accuracy, and speeds up the hiring process.

  - 📄 Resumes and job descriptions are uploaded to S3 and automatically processed by AWS Lambda.
  - 🔍 AWS Lambda functions to extract text, clean the data, and store structured information in Amazon DynamoDB.
  - 🧮 The matching engine in ECS Fargate performs cosine similarity search between resumes and job descriptions.
  - 📊 Ranked candidate results are returned through the Streamlit app via the Application Load Balancer.


### 🏗️ Architecture


This project follows a serverless architecture.
Resumes and job descriptions are uploaded through a Streamlit UI running on ECS Fargate, processed through Lambda, stored in DynamoDB, and matched using ML models within the ECS application.

Core Components

- Amazon S3 → Stores uploaded resumes and job descriptions

- AWS Lambda → Extracts and preprocesses text from uploaded documents

- Amazon DynamoDB → Stores cleaned text, metadata, and match results

- Amazon ECS (Fargate) → Runs the Streamlit UI and ML matching engine

- Amazon ECR → Stores container images

- Application Load Balancer → Provides public access

- CloudWatch → Monitors logs and application performance

### 🏗️Architecture Diagram

![cloud architecture diagram](https://github.com/user-attachments/assets/f21c24a5-6001-45c1-a746-eb2692c62b16)



## 🔌 API Endpoints (Internal Routes)

Since the project does not use API Gateway, these are logical internal endpoints handled inside the Streamlit UI:

### Endpoint	Description

- /upload_resumes/ -	  Uploads resume to S3 and triggers Lambda

- /upload_jobs/	-     Uploads job description to S3

- /processed match/	-        Runs similarity computation

- /match results table/{job_id} -	  Displays ranked resumes

- /resumes table/	-      Retrieves processed resume data

- /jobs table/  -   	Retrieves job description data

### 🔄 Data Flow

### 📁 Document Upload Flow

- User uploads documents through Streamlit UI (ECS).
- Files are stored in S3.
- S3 triggers Lambda to start processing.
- Lambda extracts text, cleans data, and saves metadata to DynamoDB.

### 🧠 Matching & Results Flow

- ECS application retrieves processed documents from DynamoDB.
- ML engine converts them into TF-IDF and BERT embeddings.
- Cosine similarity is computed for each resume–job pair.
- Ranked results are displayed in the UI.

### 📌 Technology Stack


### Frontend & Application Layer:
1. Amazon ECS with AWS Fargate: The Streamlit-based web application runs as containerized tasks in Amazon Elastic Container Service using Fargate serverless compute. This eliminates the need to manage underlying servers while providing automatic scaling based on demand.
  
2. Application Load Balancer (ALB): An internet-facing Application Load Balancer distributes incoming HTTP traffic across multiple ECS tasks running in different availability zones, ensuring high availability and fault tolerance.

### Storage Layer:
1. Amazon S3: All uploaded resumes and job descriptions are stored in separate S3 buckets with versioning enabled. The buckets (jobs for uploaded job descriptions, resumes for uploaded resumes and processed for matched results) provide durable, scalable object storage with encryption at rest.
   
2. Amazon DynamoDB: A NoSQL database stores processed resume metadata, job descriptions, and match results in three tables: ResumesTable, JobsTable, and MatchResultsTable. DynamoDB's on-demand pricing and automatic scaling handle variable workloads efficiently.

### Backend layer:
1. AWS Lambda: Serverless functions automatically process newly uploaded files from S3. When a resume is uploaded, Lambda functions are triggered via S3 event notifications to extract text, clean data, and store structured information in DynamoDB.
   
2. Python (NLTK, spaCy, PyPDF2, python-docx) – Used for text extraction and NLP preprocessing
 
3. BERT Embeddings – For deep semantic understanding
   
4. TF-IDF (Scikit-learn) – For term-weighting and vectorization
 
5. Cosine Similarity – For resume–job matching scores


### Container Registry
1. Amazon ECR (Elastic Container Registry): The Docker container image for the application is stored in a private ECR repository, enabling version control and secure image distribution to ECS tasks.



### 🔒 Security


### 1️⃣ Data Security

🔹 Encryption of data at Rest
- All resumes and job descriptions stored in Amazon S3 are encrypted by default.
- Processed metadata stored in DynamoDB is automatically encrypted using AWS-managed keys.
- Docker images stored in ECR are also encrypted.
  
🔹 Encryption in Transit
 - All communication between services uses HTTPS (TLS).
 - The Application Load Balancer enforces secure communication between the user and the ECS application.

### 2️⃣ Access Control & Identity Management

🔹 AWS IAM Roles & Policies
- Fine-grained IAM roles restrict every service to only the permissions it needs:
- Lambda can only read from S3 and write to DynamoDB.
- ECS tasks can only access specific tables and buckets.
- Streamlit app uses temporary role credentials (no hardcoded keys).
 - Follows Principle of Least Privilege to minimize security risk.
   
🔹 No Public Access to Internal Resources
- S3 buckets have Block Public Access enabled.
- DynamoDB tables are not publicly accessible—only allowed through IAM and VPC endpoints.
- Private subnets ensure internal services (ECS, Lambda) are shielded from public internet exposure.

  ### 3️⃣ Network Security
  
🔹 Amazon VPC Network Isolation
- ECS tasks and Lambda functions operate inside a secure VPC.
- Backend resources (DynamoDB, S3) are connected using VPC endpoints where possible.
- This prevents traffic from leaving Amazon’s internal network.
  
🔹 Security Groups
- Security groups limit inbound/outbound traffic:
- ALB accepts only HTTP/HTTPS traffic from the public internet.
- ECS tasks only accept traffic from the ALB.
- No direct external access to backend processing services.
  
🔹 Subnet Segmentation
- Public subnets host only the Load Balancer.
- Private subnets host ECS tasks and Lambda for increased protection.
  
 ### 4️⃣ Monitoring & Threat Detection
🔹 Amazon CloudWatch
- Monitors ECS tasks, memory, CPU, error logs, and Lambda failures.
- Alerts are set for abnormal behavior such as repeated errors or unusual traffic.
  
🔹 S3 Access Logs
- Tracks object-level access to detect any unauthorized activity.
  
🔹 AWS CloudTrail
- Records API calls across all services for auditing and tracking changes.



### ⚠️ Limitations

1️⃣ Resume Format Limitations
- The system performs best with PDF and DOCX files; resumes containing images, scanned pages, or non-selectable text may extract poorly or require OCR.

2️⃣ NLP Accuracy Constraints
- Matching accuracy depends on the quality of text extraction and the training of BERT.
- Highly creative or visually designed resumes may not convert into meaningful embeddings.

3️⃣ Processing Time for Large Files
- Lambda has time and memory limits (max 15 minutes), which may affect very large documents or heavy preprocessing tasks.

4️⃣ Limited Multilingual Support
- Current pipeline is optimized for English; resumes and job descriptions in other languages may produce inaccurate matches.

5️⃣ No Advanced Recruiter Feedback Loop
- The system does not learn from recruiter selections over time unless manually retrained with more data.

6️⃣ Dependence on Cloud Services
- The application is fully dependent on AWS availability and network connectivity.
- Costs can increase with heavy usage, especially for ECS Fargate and DynamoDB.

7️⃣ Semantic Gaps
- Even though BERT is powerful, it may still miss certain domain-specific skills or unconventional writing patterns.
- Costs can increase with heavy usage, especially for ECS Fargate and DynamoDB.


### 📌 Permissions Included (IAM Roles & Policies)

Below is a clear list of the IAM permissions required for each component in the project.

1️⃣ Lambda Permissions

Lambda functions need the following permissions:

🔹 Read Access on S3

 - s3:GetObject
 - s3:ListBucket

🔹 Write Access on DynamoDB

 - dynamodb:PutItem
 - dynamodb:UpdateItem
 - dynamodb:GetItem

🔹 CloudWatch Logging

 - logs:CreateLogGroup
 - logs:CreateLogStream
 - logs:PutLogEvents

These allow Lambda to process files and store cleaned metadata securely.

2️⃣ ECS Task Execution Role Permissions

🔹 Pull Container Images from ECR

- ecr:GetDownloadUrlForLayer
- ecr:BatchGetImage

🔹 CloudWatch Logs

- logs:CreateLogStream
- logs:PutLogEvents

🔹 DynamoDB Read Permissions

- dynamodb:Scan
- dynamodb:GetItem
- dynamodb:Query

These allow the ECS container to retrieve data and perform similarity computation.

3️⃣ S3 Bucket Permissions

S3 buckets have:

🔹 Block Public Access (Mandatory)
- Ensures no file in S3 is publicly accessible.

🔹 IAM-Restricted Access
- Only Lambda and ECS roles can:
  
- Upload files
- Read files
- Trigger events
- Ensures secure isolation of sensitive user data.

4️⃣ DynamoDB Permissions

- DynamoDB tables are private and accessible only to:
  
- Lambda function roles
- ECS task roles
  
-- Permissions include:

   - GetItem
   -  PutItem
   -  Query
   -  BatchGetItem

5️⃣ ALB & VPC Permissions

  -  Load Balancer
   - No special permissions; only exposes port 80/443 publicly.

   --VPC & Networking
    - ECS tasks use a task role and execution role.
 
   --Security Groups allow:
   
    - ALB   →   ECS traffic
    - ECS   →   DynamoDB access (through VPC endpoints)

6️⃣ Developer & Admin Permissions

Admin or developer roles may need:
- ecs:*
- lambda:*
- dynamodb:*
- s3:*
- iam: PassRole
- ecr:*
- But these should be restricted to trusted users only.


###  How Team Members Access the Project

1. **IAM User Setup**: Admin creates IAM user and attach necessary policies.
2. **AWS CLI Configuration**: Team members configure AWS CLI with provided credentials
3. **Console Access**: Login to AWS Console with IAM user credentials
4. **Development**: Full access to modify Lambda code, test APIs, view logs, and debug issues


### Screenshots:



<img width="1920" height="1020" alt="application interface" src="https://github.com/user-attachments/assets/27feeafe-7ca8-48fa-a37f-aa22145cee87" />


<img width="1920" height="1020" alt="resume matcher comparison" src="https://github.com/user-attachments/assets/f40a0612-9064-4a50-baa6-ae9741a44f3f" />


<img width="1920" height="1080" alt="Match results" src="https://github.com/user-attachments/assets/209c59b5-fcc2-4793-94f3-3c5b6b74aeb9" />


<img width="1920" height="1020" alt="Score distribution" src="https://github.com/user-attachments/assets/f6e0a895-995a-4dda-80b4-a145b5f5c224" />


<img width="1920" height="1080" alt="ranked candidate score" src="https://github.com/user-attachments/assets/0eeac27f-6769-4204-a930-1aa2f725f383" />

