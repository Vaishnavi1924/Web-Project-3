# Web-Project-3
AI-Powered Applicant Tracking System (ATS)
Day 1: Set up the project
Open the VS Code terminal and run:
mkdir ai-ats
cd ai-ats
mkdir frontend, backend, ai-service


Create the React application:
npm create vite@latest frontend -- --template react


Install the frontend libraries:
cd frontend
npm install
npm install @mui/material @emotion/react @emotion/styled
npm install @mui/icons-material @tanstack/react-query axios react-router-dom


Create the Express backend:
cd ..
cd backend
npm init -y
npm install express mongoose cors dotenv multer pdf-parse
npm install -D nodemon


Install the Python service dependencies later in Week 2.
Day 2: Configure Express
Create backend/src/server.js:
const express = require("express");
const cors = require("cors");
require("dotenv").config();

const app = express();

app.use(cors({
  origin: process.env.FRONTEND_URL || "http://localhost:5173"
}));
app.use(express.json());

app.get("/api/health", (req, res) => {
  res.json({ status: "ok", message: "ATS API is running" });
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`ATS API running on port ${PORT}`);
});


In backend/package.json, add these scripts:
"scripts": {
  "start": "node src/server.js",
  "dev": "nodemon src/server.js"
}


Create backend/.env:
PORT=5000
FRONTEND_URL=http://localhost:5173
MONGODB_URI=mongodb://127.0.0.1:27017/ai_ats


Day 3: Connect MongoDB
Create backend/src/config/db.js:
const mongoose = require("mongoose");

async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log("MongoDB connected successfully");
  } catch (error) {
    console.error("MongoDB connection failed:", error.message);
    process.exit(1);
  }
}

module.exports = connectDB;


Update server.js to connect to the database before listening:
const connectDB = require("./config/db");

async function startServer() {
  await connectDB();

  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`ATS API running on port ${PORT}`);
  });
}

startServer();
