from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import anthropic
import httpx
from bs4 import BeautifulSoup
import os

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

client = anthropic.Anthropic(api_key=os.environ.get("CLAUDE_API_KEY"))

class URLRequest(BaseModel):
    url: str

@app.get("/")
def root():
    return {"status": "SEOzy Backend Live!"}

@app.post("/analyze")
async def analyze_seo(request: URLRequest):
    try:
        async with httpx.AsyncClient() as http:
            response = await http.get(request.url, timeout=10)
            html = response.text

        soup = BeautifulSoup(html, "html.parser")
        title = soup.find("title")
        meta_desc = soup.find("meta", attrs={"name": "description"})
        h1 = soup.find("h1")

        current_title = title.text if title else "Not found"
        current_desc = meta_desc["content"] if meta_desc else "Not found"
        current_h1 = h1.text if h1 else "Not found"

        message = client.messages.create(
            model="claude-opus-4-6",
            max_tokens=1000,
            messages=[{
                "role": "user",
                "content": f"""Analyze this website SEO data and give improvements:
Title: {current_title}
Meta Description: {current_desc}
H1: {current_h1}
URL: {request.url}

Reply in JSON format only:
{{
  "seo_score": 75,
  "improved_title": "better title here",
  "improved_description": "better description here",
  "issues": ["issue 1", "issue 2"],
  "schema": {{"@context": "https://schema.org", "@type": "WebPage"}}
}}"""
            }]
        )

        import json
        result = json.loads(message.content[0].text)
        result["current_title"] = current_title
        result["current_description"] = current_desc
        return result

    except Exception as e:
        return {"error": str(e)}
