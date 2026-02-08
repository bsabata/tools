# SDK & API Examples

## Python SDK: Generate a Slide

```python
from google import genai
from google.genai import types

client = genai.Client()

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=["Create a professional presentation slide with title 'Q4 Results' "
              "showing revenue: $4.2M (+15%), users: 50K (+22%), NPS: 72 (+5). "
              "Use a dark theme with green accent for positive metrics. "
              "Clean corporate design. 16:9 widescreen."],
    config=types.GenerateContentConfig(
        response_modalities=['TEXT', 'IMAGE'],
        image_config=types.ImageConfig(
            aspect_ratio="16:9",
            image_size="2K"
        )
    )
)

for part in response.parts:
    if part.text is not None:
        print(part.text)
    elif part.inline_data is not None:
        image = part.as_image()
        image.save("slide_q4_results.png")
```

## Python SDK: Generate a Multi-Slide Deck

```python
from google import genai
from google.genai import types

client = genai.Client()

style = ("Color scheme: deep navy primary, white text, gold accent. "
         "Clean sans-serif typography. Dark gradient background. "
         "Professional consistent margins. 16:9 widescreen.")

slides = [
    f"{style} Title slide: 'Machine Learning 101'. Subtitle: 'An Introduction'. "
    f"Author: 'Data Science Team'. Clean, modern.",

    f"{style} Bullet point slide. Header: 'What is ML?'. Bullets: "
    f"'Subset of AI', 'Learns from data', 'Improves with experience', "
    f"'Powers modern applications'.",

    f"{style} Comparison slide. Title: 'Supervised vs Unsupervised'. "
    f"Left: 'Supervised - Labeled data, Classification, Regression'. "
    f"Right: 'Unsupervised - Unlabeled data, Clustering, Dimensionality reduction'.",

    f"{style} Conclusion slide. Large text: 'Thank You'. "
    f"Subtitle: 'Questions? team@company.com'."
]

for i, prompt in enumerate(slides, 1):
    response = client.models.generate_content(
        model="gemini-3-pro-image-preview",
        contents=[prompt],
        config=types.GenerateContentConfig(
            response_modalities=['TEXT', 'IMAGE'],
            image_config=types.ImageConfig(
                aspect_ratio="16:9",
                image_size="2K"
            )
        )
    )
    for part in response.parts:
        if part.inline_data is not None:
            image = part.as_image()
            image.save(f"slide_{i:02d}.png")
            print(f"Saved: slide_{i:02d}.png")
```

## JavaScript SDK: Generate a Slide

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});

const response = await ai.models.generateContent({
    model: "gemini-3-pro-image-preview",
    contents: "Create a professional slide with title 'Project Roadmap' showing " +
              "Q1: Research, Q2: Prototype, Q3: Beta Launch, Q4: GA Release. " +
              "Timeline format with arrows between phases. Blue gradient background. 16:9.",
    config: {
        responseModalities: ['TEXT', 'IMAGE'],
        imageConfig: {
            aspectRatio: "16:9",
            imageSize: "2K"
        }
    }
});

for (const part of response.candidates[0].content.parts) {
    if (part.text) {
        console.log(part.text);
    } else if (part.inlineData) {
        const buffer = Buffer.from(part.inlineData.data, "base64");
        fs.writeFileSync("slide_roadmap.png", buffer);
        console.log("Saved: slide_roadmap.png");
    }
}
```

## REST API (curl) Quick Reference

```bash
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Create a clean title slide: \"Quarterly Review Q4 2025\". Subtitle: \"Finance Department\". Dark blue background, white text. Professional. 16:9."}]
    }],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {
        "aspectRatio": "16:9",
        "imageSize": "2K"
      }
    }
  }' | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
for part in data['candidates'][0]['content']['parts']:
    if 'inlineData' in part:
        with open('slide_output.png', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print('Saved: slide_output.png')
"
```

**Important:** This inline JSON approach is only safe for very short prompts without special characters. For longer prompts, prompts with quotes or special characters, or any production use, always use the file-based approach described in the core SKILL.md workflow (write JSON to a temp file with `mktemp` and use `curl -d @file.json`).

## Dependencies

For Python SDK usage:
```bash
pip install google-genai pillow
```

For JavaScript SDK usage:
```bash
npm install @google/genai
```

For the curl-based workflow (default):
- `curl` (pre-installed on macOS and Linux)
- `python3` (for JSON handling and base64 decoding)
