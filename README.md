# SDA2007-Waad Abadil Saed Alharthi
# Week-10 Assignment

# Part 1: MongoDB Atlas Configuration (Optional):
  - Create a MongoDB Atlas account and free cluster.
  - Allow EC2 instance IP in Network Access.
  - Create a database user and obtain the connection string.


# Part 2: S3 Bucket for Frontend:
1. Create an S3 bucket named waad-blogapp-frontend in the eu-north-1 region.
2. Disable Block all public access.
3. Enable Static website hosting with index.html as the index document.
4. Add the following bucket policy to allow public read access:

   json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::waad-blogapp-frontend/*"
    }
  ]
}

5. Build the frontend and deploy files to this bucket.


#part3: S3 Bucket for Media Uploads:

1. Create S3 bucket named waad-blogapp-media.
2. Disable Block all public access.
3. Configure CORS with the following settings:

Json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]

4. Test uploading and retrieving files from this bucket: it is working.
https://waad-blogapp-media.s3.eu-north-1.amazonaws.com/photo_5805592218847790348_y.jpg


# Part4: IAM User and Policy for Media Bucket Access:

1. Create an IAM user named waad-app-user with programmatic access.
2. Create and attach this custom policy:{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::waad-blogapp-media",
        "arn:aws:s3:::waad-blogapp-media/*"
      ]
    }
  ]
}
3. Save the Access Key ID and Secret Access Key securely.


# Part 5: EC2 Backend Setup

1. Launch a t3.micro EC2 instance with Ubuntu 22.04 LTS in eu-north-1.
2. Configure Security Group to allow ports:
   - SSH (22)
   - HTTP (80)
   - HTTPS (443)
   - Custom TCP (5000)
3. SSH into the instance and run the following User Data script or execute commands manually:

#!/bin/bash
apt update -y
apt install -y git curl unzip tar gcc g++ make unzip

su - ubuntu << 'EOF'
export NVM_DIR="$HOME/.nvm"
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source "$NVM_DIR/nvm.sh"
nvm install --lts
nvm use --lts
npm install -g pm2
EOF

curl -L https://downloads.mongodb.com/compass/mongosh-2.1.1-linux-x64.tgz -o mongosh.tgz
tar -xvzf mongosh.tgz
mv mongosh-*/bin/mongosh /usr/local/bin/
chmod +x /usr/local/bin/mongosh
rm -rf mongosh*

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
./aws/install
rm -rf aws awscliv2.zip

4. Clone the project repository:

git clone https://github.com/cw-barry/blog-app-MERN.git /home/ubuntu/blog-app

# Navigate to the project directory
cd /home/ubuntu/blog-app

5. Configure backend .env file with credentials and settings:

cd /home/ubuntu/blog-app/backend

cat > .env << EOF
PORT=5000
HOST=0.0.0.0
MODE=production

MONGODB=mongodb+srv://test:qazqwe123@mongodb.txkjsso.mongodb.net/blog-app

JWT_SECRET=$(openssl rand -hex 32)
JWT_EXPIRE=30min
JWT_REFRESH=$(openssl rand -hex 32)
JWT_REFRESH_EXPIRE=3d

AWS_ACCESS_KEY_ID=<aws-access-key-id>
AWS_SECRET_ACCESS_KEY=<your-aws-secret-access-key>
AWS_REGION=eu-north-1
S3_BUCKET=waad-blogapp-media

MEDIA_BASE_URL=https:https://waad-blogapp-media.s3.eu-north-1.amazonaws.com/photo_5805592218847790348_y.jpg

DEFAULT_PAGINATION=20
EOF

6. Configure frontend .env:
cd ../frontend

cat > .env << EOF
VITE_BASE_URL=http://waad-blogapp-frontend.s3-website.eu-north-1.amazonaws.com
VITE_MEDIA_BASE_URL=https://waad-blogapp-media>.eu-north-1.amazonaws.com
EOF

7. Configure AWS CLI credentials on EC2:
aws configure
- IAM user credentials created earlier.
- Set region to eu-north-1.
- Set output format to json.

8. Install backend dependencies and start backend server:
cd /home/ubuntu/blog-app/backend
npm install
mkdir -p logs
pm2 start index.js waad-blog-backend
pm2 save
pm2 startup

9. Build and deploy frontend to S3:
cd /home/ubuntu/blog-app/frontend
npm install -g pnpm@latest-10
pnpm install
pnpm run build
aws s3 sync dist/ s3://waad-blogapp-frontend






