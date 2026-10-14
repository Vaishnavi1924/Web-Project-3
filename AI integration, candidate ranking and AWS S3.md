Day 1: Create the matching service
Create backend/src/services/matching.js:
function calculateMatch(resumeText, requiredSkills) {
  const text = resumeText.toLowerCase();

  const skills = [...new Set(
    requiredSkills.map(skill => skill.trim()).filter(Boolean)
  )];

  const matchedSkills = skills.filter(skill =>
    text.includes(skill.toLowerCase())
  );

  const missingSkills = skills.filter(skill =>
    !text.includes(skill.toLowerCase())
  );

  const score = skills.length
    ? Math.round((matchedSkills.length / skills.length) * 100)
    : 0;

  return {
    score,
    matchedSkills,
    missingSkills,
    method: "keyword-baseline"
  };
}

module.exports = { calculateMatch };


This first version provides a transparent baseline score based on the percentage of required skills found in the resume. It is not a complete semantic similarity score.
Day 2: Rank applications for each job
Add this endpoint to backend/src/routes/applications.js:
const { calculateMatch } = require("../services/matching");

// Place this route before any /:id routes.
router.get("/job/:jobId/ranked", async (req, res) => {
  try {
    const job = await Job.findById(req.params.jobId);

    if (!job) {
      return res.status(404).json({ message: "Job not found" });
    }

    const applications = await Application.find({
      jobId: job._id
    });

    const ranked = applications.map(application => {
      const match = calculateMatch(
        application.resumeText || "",
        job.requiredSkills || []
      );
