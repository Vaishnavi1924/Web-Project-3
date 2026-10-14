Day 3: Extract text from PDF resumes
Create backend/src/services/resume.js:
const pdfParse = require("pdf-parse");

async function extractResumeText(buffer) {
  const result = await pdfParse(buffer);
  const text = result.text.trim();

  if (!text) {
    throw new Error(
      "No readable text found. OCR is needed for scanned PDFs."
    );
  }

  return text;
}

const KNOWN_SKILLS = [
  "JavaScript",
  "React",
  "Node.js",
  "Express",
  "MongoDB",
  "Python",
  "Java",
  "SQL",
  "HTML",
  "CSS",
  "Machine Learning",
  "AWS",
  "Docker",
  "Git",
  "REST API"
];

function extractSkills(text) {
  const normalized = text.toLowerCase();

  return KNOWN_SKILLS.filter(skill =>
    normalized.includes(skill.toLowerCase())
  );
}

module.exports = {
  extractResumeText,
  extractSkills
};


This is a baseline extractor. More advanced extraction can use a language model or a dedicated NLP model to recognize additional skills and related terms.
Day 4: Create the application API
Create backend/src/routes/applications.js:
const express = require("express");
const Application = require("../models/Application");
const Job = require("../models/Job");
const upload = require("../middleware/upload");

const {
  extractResumeText,
  extractSkills
} = require("../services/resume");

const router = express.Router();

router.post("/", upload.single("resume"), async (req, res) => {
  try {
    const { jobId, candidateName, email } = req.body;

    if (!req.file) {
      return res.status(400).json({
        message: "Please upload a PDF resume"
      });
    }
