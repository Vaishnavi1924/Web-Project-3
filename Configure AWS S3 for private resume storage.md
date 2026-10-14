Day 4: Configure AWS S3 for private resume storage
Install the AWS SDK:
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner


Create backend/src/services/storage.js:
const {
  S3Client,
  PutObjectCommand,
  GetObjectCommand
} = require("@aws-sdk/client-s3");

const {
  getSignedUrl
} = require("@aws-sdk/s3-request-presigner");

const s3 = new S3Client({
  region: process.env.AWS_REGION
});

async function uploadResume(buffer, key) {
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET,
    Key: key,
    Body: buffer,
    ContentType: "application/pdf"
  }));

  return key;
}

async function createResumeDownloadUrl(key) {
  return getSignedUrl(
    s3,
    new GetObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key
    }),
    { expiresIn: 300 }
  );
}

module.exports = {
  uploadResume,
  createResumeDownloadUrl
};


Configure the backend environment:
AWS_REGION=ap-south-1
S3_BUCKET=your-private-bucket-name
