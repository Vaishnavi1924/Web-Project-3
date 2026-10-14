Install the OpenAI SDK:
cd backend
npm install openai


Create backend/src/services/aiMatching.js:
const OpenAI = require("openai");

async function explainMatch(resumeText, job) {
  if (!process.env.OPENAI_API_KEY) {
    return {
      enabled: false,
      message: "AI explanation is not configured"
    };
  }

  const client = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const response = await client.responses.create({
    model: process.env.OPENAI_MODEL || "gpt-4o-mini",
    instructions: [
    "You assist recruiters with job-related resume analysis.",
      "Return valid JSON only.",
      "Evaluate evidence for the required skills.",
      "Do not infer skills that are not supported by the resume.",
      "Ignore names, photos, age, gender, religion and other",
      "protected or irrelevant personal characteristics.",
      "Do not make a hiring decision."
    ].join(" "),
    input: JSON.stringify({
      jobTitle: job.title,
      jobDescription: job.description,
      requiredSkills: job.requiredSkills,
      resumeText: resumeText.slice(0, 15000),
      requestedOutput: {
        evidence: [
          {
            skill: "string",
            found: "boolean",
            explanation: "string"
          }
        ],
        summary: "string"
      }
    })
  });

  return {
    enabled: true,
    analysis: response.output_text
  };
}

module.exports = { explainMatch };
