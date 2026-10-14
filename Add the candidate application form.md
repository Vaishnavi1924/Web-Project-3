Day 3: Add the candidate application form
Create frontend/src/pages/Apply.jsx:
import { useState } from "react";
import { useMutation, useQuery } from "@tanstack/react-query";
import {
  Alert,
  Button,
  MenuItem,
  Stack,
  TextField,
  Typography
} from "@mui/material";
import api from "../api/client";

export default function Apply() {
  const [jobId, setJobId] = useState("");
  const [candidateName, setCandidateName] = useState("");
  const [email, setEmail] = useState("");
  const [resume, setResume] = useState(null);
  const [message, setMessage] = useState("");

  const jobs = useQuery({
    queryKey: ["jobs"],
    queryFn: async () => (await api.get("/jobs")).data
  });

  const apply = useMutation({
    mutationFn: async () => {
      const data = new FormData();
      data.append("jobId", jobId);
      data.append("candidateName", candidateName);
      data.append("email", email);
      data.append("resume", resume);

      return api.post("/applications", data);
    },
    onSuccess: () => {
      setMessage("Application submitted successfully.");
    },
    onError: err => {
      setMessage(err.response?.data?.message || "Submission failed.");
    }
  });

  function handleSubmit(event) {
    event.preventDefault();

    if (!resume || resume.type !== "application/pdf") {
      setMessage("Please select a PDF resume.");
      return;
    }

    setMessage("");
    apply.mutate();
  }

  return (
    <Stack
      component="form"
      spacing={2}
      onSubmit={handleSubmit}
      sx={{ maxWidth: 600, mx: "auto", p: 3 }}
    >
      <Typography variant="h4">Apply for a job</Typography>

      <TextField
        select
        label="Select job"
        value={jobId}
        onChange={e => setJobId(e.target.value)}
        required
      >
        {(jobs.data || []).filter(job => job.status === "Open").map(job => (
          <MenuItem key={job._id} value={job._id}>
            {job.title}
          </MenuItem>
        ))}
      </TextField>

      <TextField
        label="Full name"
        value={candidateName}
        onChange={e => setCandidateName(e.target.value)}
        required
      />

      <TextField
        label="Email"
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        required
      />

      <Button variant="outlined" component="label">
        Select PDF resume
        <input
          hidden
          type="file"
          accept=".pdf,application/pdf"
          onChange={e => setResume(e.target.files?.[0] || null)}
        />
      </Button>

      {resume && <Typography>{resume.name}</Typography>}

      {message && (
        <Alert severity={message.includes("successfully") ? "success" : "info"}>
          {message}
        </Alert>
      )}

      <Button
        type="submit"
        variant="contained"
        disabled={apply.isPending || !resume}
      >
        {apply.isPending ? "Submitting..." : "Submit application"}
      </Button>
    </Stack>
  );
}
});

  const apply = useMutation({
    mutationFn: async () => {
      const data = new FormData();
      data.append("jobId", jobId);
      data.append("candidateName", candidateName);
      data.append("email", email);
      data.append("resume", resume);

      return api.post("/applications", data);
    },
    onSuccess: () => {
      setMessage("Application submitted successfully.");
    },
    onError: err => {
      setMessage(err.response?.data?.message || "Submission failed.");
    }
  });

  function handleSubmit(event) {
    event.preventDefault();

    if (!resume || resume.type !== "application/pdf") {
      setMessage("Please select a PDF resume.");
      return;
    }

    setMessage("");
    apply.mutate();
  }

  return (
    <Stack
      component="form"
      spacing={2}
      onSubmit={handleSubmit}
      sx={{ maxWidth: 600, mx: "auto", p: 3 }}
    >
      <Typography variant="h4">Apply for a job</Typography>

      <TextField
        select
        label="Select job"
        value={jobId}
        onChange={e => setJobId(e.target.value)}
        required
      >
        {(jobs.data || []).filter(job => job.status === "Open").map(job => (
          <MenuItem key={job._id} value={job._id}>
            {job.title}
          </MenuItem>
        ))}
      </TextField>

      <TextField
        label="Full name"
        value={candidateName}
        onChange={e => setCandidateName(e.target.value)}
        required
      />

      <TextField
        label="Email"
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        required
      />

      <Button variant="outlined" component="label">
        Select PDF resume
        <input
          hidden
          type="file"
          accept=".pdf,application/pdf"
          onChange={e => setResume(e.target.files?.[0] || null)}
        />
      </Button>

      {resume && <Typography>{resume.name}</Typography>}

      {message && (
        <Alert severity={message.includes("successfully") ? "success" : "info"}>
          {message}
        </Alert>
      )}

      <Button
        type="submit"
        variant="contained"
        disabled={apply.isPending || !resume}
      >
        {apply.isPending ? "Submitting..." : "Submit application"}
      </Button>
    </Stack>
  );
}


The backend must remain responsible for enforcing the file-size limit, PDF validation and job eligibility. Frontend validation alone is not a security control.
Day 4: Build the recruiter dashboard
Create a dashboard that displays:
- Total job openings.
- Total applications.
- Candidates awaiting review.
- Shortlisted candidates.
- Ranked applications with matched and missing skills.
- Status update controls.
- Resume review links restricted to authorized recruiters.
Use MUI components such as Card, Grid, Table, Chip, Select, MenuItem, and CircularProgress.
The starter ZIP already contains dashboard and application pages to build on. Connect those pages to /api/applications and /api/applications/job/:jobId/ranked using useQuery. Use useMutation for status changes and invalidate the application query after each successful update.
Day 5: Testing, authentication and final integration
Complete the following before your final demonstration:
1. Add registration and login endpoints, hash passwords with bcrypt, and use secure authentication.
2. Protect recruiter-only APIs with middleware and verify role permissions on the backend.
3. Validate job IDs, email addresses, request sizes, file contents and status transitions.
4. Test failed MongoDB connections, invalid PDFs, empty resumes and unavailable AI services.
5. Test the complete workflow: create a job → submit a resume → extract skills → calculate the score → view ranked applicants → update status.
6. Configure S3 access controls, privacy and document-retention rules.
7. Document installation steps, API routes, environment variables and screenshots for your project report.
