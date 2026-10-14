Week 2 — Resume upload, PDF parsing and skill extraction

Goal: Allow candidates to apply for jobs, upload PDF resumes, extract text and save application records.
Day 1: Create the Application model
Create backend/src/models/Application.js:
const mongoose = require("mongoose");

const applicationSchema = new mongoose.Schema(
  {
    jobId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Job",
      required: true
    },
    candidateName: {
      type: String,
      required: true,
      trim: true
    },
    email: {
      type: String,
      required: true,
      lowercase: true,
      trim: true
    },
    resumeUrl: String,
    resumeText: String,
    extractedSkills: {
      type: [String],
      default: []
    },
    matchScore: {
      type: Number,
      default: 0,
      min: 0,
      max: 100
    },
    matchedSkills: [String],
    missingSkills: [String],
    status: {
      type: String,
      enum: [
        "Applied",
        "Reviewed",
        "Shortlisted",
        "Interview",
        "Rejected",
        "Hired"
      ],
      default: "Applied"
    }
  },
  { timestamps: true }
);

module.exports = mongoose.model("Application", applicationSchema);


Day 2: Configure Multer for PDF uploads
Create backend/src/middleware/upload.js:
const multer = require("multer");

const storage = multer.memoryStorage();

function fileFilter(req, file, callback) {
  if (file.mimetype !== "application/pdf") {
    return callback(new Error("Only PDF resumes are allowed"));
  }

  callback(null, true);
}

const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024
  }
});

module.exports = upload;
