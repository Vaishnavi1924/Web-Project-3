Week 4 — React dashboard, React Query, MUI and final testing
Goal: Complete the user interface and connect all the services into one usable application.
Day 1: Configure React Query
Create frontend/src/main.jsx:
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { CssBaseline, ThemeProvider, createTheme } from "@mui/material";
import App from "./App";

const queryClient = new QueryClient();

const theme = createTheme({
  palette: {
    mode: "light",
    primary: { main: "#2563eb" },
    background: { default: "#f5f7fb" }
  },
  shape: { borderRadius: 10 }
});

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        <App />
      </ThemeProvider>
    </QueryClientProvider>
  </React.StrictMode>
);


This provides a shared query cache and Material UI theme to the whole application.
Day 2: Connect the frontend to the backend
Create frontend/.env:
VITE_API_URL=http://localhost:5000/api


Create frontend/src/api/client.js:
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL ||
    "http://localhost:5000/api"
});

export default api;


Create frontend/src/pages/Jobs.jsx:
import { useState } from "react";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import {
  Alert,
  Button,
  Card,
  CardContent,
  Stack,
  TextField,
  Typography
} from "@mui/material";
import api from "../api/client";

export default function Jobs() {
  const queryClient = useQueryClient();
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [skills, setSkills] = useState("");
  const [error, setError] = useState("");

  const jobsQuery = useQuery({
    queryKey: ["jobs"],
    queryFn: async () => (await api.get("/jobs")).data
  });

  const createJob = useMutation({
    mutationFn: async job => (await api.post("/jobs", job)).data,
    onSuccess: () => {
      setTitle("");
      setDescription("");
      setSkills("");
      setError("");
      queryClient.invalidateQueries({ queryKey: ["jobs"] });
    },
    onError: err => {
      setError(err.response?.data?.message || "Unable to create job");
    }
  });

  function handleSubmit(event) {
    event.preventDefault();
    setError("");

    createJob.mutate({
      title,
      description,
      requiredSkills: skills.split(",").map(s => s.trim()).filter(Boolean)
    });
  }

  return (
    <Stack spacing={3} sx={{ maxWidth: 900, mx: "auto", p: 3 }}>
      <Typography variant="h4">Manage Jobs</Typography>

      <Card>
        <CardContent>
          <Stack component="form" spacing={2} onSubmit={handleSubmit}>
            <Typography variant="h6">Create a job opening</Typography>

            <TextField
              label="Job title"
              value={title}
              onChange={e => setTitle(e.target.value)}
              required
            />

            <TextField
              label="Job description"
              value={description}
              onChange={e => setDescription(e.target.value)}
              multiline
              minRows={3}
              required
            />

            <TextField
              label="Required skills, separated by commas"
              value={skills}
              onChange={e => setSkills(e.target.value)}
              placeholder="React, Node.js, MongoDB"
            />

            {error && <Alert severity="error">{error}</Alert>}

            <Button
              type="submit"
              variant="contained"
              disabled={createJob.isPending}
            >
              {createJob.isPending ? "Saving..." : "Create job"}
            </Button>
          </Stack>
        </CardContent>
      </Card>

      {jobsQuery.isPending && <Typography>Loading jobs...</Typography>}
      {jobsQuery.isError && (
        <Alert severity="error">Could not load jobs. Check the backend.</Alert>
      )}

      {(jobsQuery.data || []).map(job => (
        <Card key={job._id}>
          <CardContent>
            <Typography variant="h6">{job.title}</Typography>
            <Typography color="text.secondary">
              {job.description}
            </Typography>
            <Typography sx={{ mt: 1 }}>
              Required skills: {(job.requiredSkills || []).join(", ")}
            </Typography>
          </CardContent>
        </Card>
      ))}
    </Stack>
  );
}
