Day 5: Add the Python microservice
Create ai-service/main.py:
from fastapi import FastAPIfrom pydantic import BaseModelapp = FastAPI(title="ATS Resume Analysis Service")class MatchRequest(BaseModel):    resume_text: str    required_skills: list[str]@app.get("/health")def health():    return {"status": "ok", "service": "ats-ai-service"}@app.post("/analyze")def analyze(request: MatchRequest):    resume = request.resume_text.lower()    skills = list(dict.fromkeys(        skill.strip()        for skill in request.required_skills        if skill.strip()    ))    matched = [        skill for skill in skills        if skill.lower() in resume    ]    missing = [        skill for skill in skills        if skill.lower() not in resume    ]



Create ai-service/requirements.txt:
fastapi
uvicorn[standard]
pydantic


Install and start it:
cd ai-service
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
