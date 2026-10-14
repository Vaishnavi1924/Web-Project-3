Day 5: Store scores and application statuses
When a resume is submitted, calculate the baseline score and store it with the application:
const match = calculateMatch(
  resumeText,
  job.requiredSkills || []
);

const application = await Application.create({
  jobId,
  candidateName,
  email,
  resumeText,
  extractedSkills,
  matchScore: match.score,
  matchedSkills: match.matchedSkills,
  missingSkills: match.missingSkills
});


Also implement the status-update endpoint:
router.patch("/:id/status", async (req, res) => {
  const allowedStatuses = [
    "Applied",
    "Reviewed",
    "Shortlisted",
    "Interview",
    "Rejected",
    "Hired"
  ];

  if (!allowedStatuses.includes(req.body.status)) {
    return res.status(400).json({
      message: "Invalid application status"
    });
  }

  try {
    const application = await Application.findByIdAndUpdate(
      req.params.id,
      { status: req.body.status },
      { new: true, runValidators: true }
