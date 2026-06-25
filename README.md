# WiCSE Shadowing -- PRISM E-Commerce Platform
A full-stack e-commerce platform build as a personal project under WiCSE's Shadowing with American Express (AMEX). This platform demonstrates real-world cloud messaging patterns using AWS services (SQS, SNS, SES) and was originally deployed on live AWS infrastructure and runnable locally via LocalStack.

# Demo
[Demo Video](https://drive.google.com/file/d/1ex_Ig3-gxTlB5eqwdu8NiR9Ql0PQABBm/view?usp=sharing)

# Features
- Browse and purchase products through a Next.js storefront
- Order confirmation flow powered by event-driven messaging pipeline
- Automated email notifications sent to customer upon order placement
- AWS-compatible cloud architecture (SQS -> SNS -> SES) simulated locally via LocalStack

# Architecture
When an order is placed, this following pipeline is run:

```
Customer Places Order
         |
         ▼
SNS Topic publishes order event
         |
         ▼
SQS Queue receives the message
         |
         ▼
Email Worker (emailWorker.ts) polls the queue
         |
         ▼
SES sends order confirmation to customer
```

# Tech Stack
| Layer | Technology |
|---|---|
| Frontend | Next.js React (Typescript) |
| Backend & APIs | Node.js |
| Database | PostgreSQL |
| Database Client | node-postgres (pg) |
| Styling | CSS / Tailwind CSS |
| Message Queue | AWS SQS (LocalStack) |
| Pub/Sub Notifications | AWS SNS (LocalStack) |
| Email Service | AWS SES (LocalStack) |
| Local AWS Emulation | LocalStack (Docker) |
| Containerization | Docker / Docker Compose |

# To Start
### Prerequisites
Node.js, Docker, Docker Compose

### 1. Clone the repository
```bash
git clone https://github.com/leonnaxie/WiSCE-Shadowing-Commerce-Platform.git
cd WiCSE-Shadowing-Commerce-Platform
```

### 2. Install dependencies
```bash
npm install
```

### 3. Start LocalStack
This will create an AWS environment with SQS, SNS and SES on port `4566`:
```bash
docker-compose up -d
```

### 4. Set up AWS Resources in LocalStack
You need to create the SNS topic, SQS queue, and SES email identity in LocalStack before running the app. Use the AWS CLI pointed at LocalStack:
```bash
# Configure a dummy local profile that doesn't need real credentials
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1

# Create SQS queue
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name email-notif-queue

# Create the SNS topic
aws --endpoint-url=http://localhost:4566 sns create-topic --name order-confirmations
 
# Subscribe the SQS queue to the SNS topic
aws --endpoint-url=http://localhost:4566 sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:000000000000:order-confirmations \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:000000000000:email-notif-queue
 
# Verify a sender email address for SES
aws --endpoint-url=http://localhost:4566 ses verify-email-identity \
  --email-address yoursordummy@gmail.com
```

### 5. Start the email worker
In a separate terminal, run the background worker that polls SQS and will send emails via the SES:
```bash
npx ts-node emailWorker.ts
```

### 6. Run the development server
```bash
npm run dev
```
You can open [http://localhost:3000](http://localhost:3000) in your browser.


# Original AWS Deployment
 
This project was originally deployed on live AWS infrastructure using the same SQS → SNS → SES pipeline. The demo video above shows the fully functioning AWS-backed version. The current codebase points to LocalStack endpoints so anyone can run it locally without an AWS account.
 
---
 
# Project Structure
 
```
├── app/                  # Next.js app directory (pages, API routes, components)
├── db/                   # Database schema / seed files
├── public/               # Static assets
├── emailWorker.ts        # Background worker: polls SQS and sends emails via SES
├── docker-compose.yml    # LocalStack setup (SNS, SQS, SES on port 4566)
├── next.config.ts        # Next.js configuration
└── tsconfig.json         # TypeScript configuration
```
 
---
 
# License
 
This project was built for personal learning and portfolio purposes.


