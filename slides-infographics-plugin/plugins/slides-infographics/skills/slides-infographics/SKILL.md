---
name: Slides & Infographics
description: Create presentation slides, infographics, timelines, comparison charts, process flows, and visual decks using Gemini's Nano Banana Pro model (gemini-3-pro-image-preview). Use this skill when the user asks you to create slides, presentations, infographics, visual summaries, timelines, data visualizations, diagrams, concept maps, or any slide/presentation-related visual.
allowed-tools: Read, Write, Bash, WebFetch
---

# Slides & Infographics

This skill creates presentation slides, infographics, and visual decks using Google's Gemini Nano Banana Pro model (`gemini-3-pro-image-preview`).

## IMPORTANT: Setup Required

Before using this skill, the user must set the `GEMINI_API_KEY` environment variable:

1. Get a free API key from [Google AI Studio](https://aistudio.google.com/)
2. Export the key in your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):
   ```bash
   export GEMINI_API_KEY="your_api_key_here"
   ```
3. Restart your terminal or run `source ~/.zshrc` (or `~/.bashrc`)

**The skill will not work without this configuration.**

## Pre-flight Check

Before making any API call, verify the key is set:

```bash
if [ -z "$GEMINI_API_KEY" ]; then
  echo "ERROR: GEMINI_API_KEY is not set. Please export it in your shell profile."
  exit 1
fi
```

If the key is missing, stop and tell the user to set it using the instructions above.

## Configuration

**Model**: `gemini-3-pro-image-preview`

**API Key**: Read from the `GEMINI_API_KEY` environment variable

**Default Settings for Slides:**
- Aspect Ratio: `16:9` (standard presentation format)
- Resolution: `2K` (2752x1536 at 16:9)
- Response Modalities: `["TEXT", "IMAGE"]`

**Recommended Settings by Visual Type:**

| Visual Type          | Aspect Ratio | Resolution | Notes                                    |
|----------------------|--------------|------------|------------------------------------------|
| Presentation Slide   | 16:9         | 2K         | Standard widescreen, matches projectors  |
| Infographic (tall)   | 9:16         | 2K         | Vertical scroll format                   |
| Infographic (wide)   | 16:9         | 2K         | Horizontal layout                        |
| Social Media Card    | 1:1          | 1K         | Square format for posts                  |
| Timeline (wide)      | 21:9         | 2K         | Ultra-wide for horizontal timelines      |
| Poster/Print         | 3:4          | 4K         | Portrait print format                    |
| Comparison Chart     | 16:9         | 2K         | Side-by-side layout                      |

## Core Workflow: Generate a Single Slide

### Step 1: Construct the slide prompt

Build a detailed prompt that includes:
- **Slide type** (title, bullet points, comparison, timeline, etc.)
- **Content** (exact text, data points, labels)
- **Visual style** (color scheme, layout, typography direction)
- **Aspect ratio** (default 16:9 for slides)

### Step 2: Write the request JSON to a temp file

**IMPORTANT:** Always use a file-based approach for the request body. This avoids "argument list too long" errors and handles special characters safely.

```bash
SLIDE_PROMPT="Create a professional presentation slide with the title 'AI in Healthcare' as large bold text centered at the top. Below the title, show three bullet points: '1. Diagnostic Accuracy', '2. Drug Discovery', '3. Patient Monitoring'. Use a modern dark blue gradient background with white text. Clean, corporate design. 16:9 widescreen layout."

# Use Python to safely construct JSON (handles special characters)
python3 -c "
import json
data = {
    'contents': [{'parts': [{'text': '''$SLIDE_PROMPT'''}]}],
    'generationConfig': {
        'responseModalities': ['TEXT', 'IMAGE'],
        'imageConfig': {
            'aspectRatio': '16:9',
            'imageSize': '2K'
        }
    }
}
with open('/tmp/gemini_slide_request.json', 'w') as f:
    json.dump(data, f)
"
```

### Step 3: Call the API

```bash
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/gemini_slide_request.json > /tmp/gemini_slide_response.json
```

### Step 4: Extract and save the slide image

```bash
python3 -c "
import json
import base64

with open('/tmp/gemini_slide_response.json') as f:
    data = json.load(f)

if 'error' in data:
    print(f'API Error: {data[\"error\"][\"message\"]}')
elif 'candidates' not in data or len(data['candidates']) == 0:
    print('No candidates returned. The prompt may have been filtered.')
else:
    for part in data['candidates'][0]['content']['parts']:
        if 'inlineData' in part:
            img_data = part['inlineData']['data']
            mime = part['inlineData']['mimeType']
            ext = 'png' if 'png' in mime else 'jpg'
            with open('slide_output.' + ext, 'wb') as out:
                out.write(base64.b64decode(img_data))
            print(f'Saved: slide_output.{ext}')
        elif 'text' in part:
            print(part['text'])
"
```

## Slide & Infographic Prompt Templates

Use these templates as the basis for constructing prompts. Fill in the bracketed placeholders with the user's content.

### Title Slide

```
Create a professional presentation title slide. The title '[TITLE]' should be
displayed as large, bold text centered on the slide. Below the title, show the
subtitle '[SUBTITLE]' in smaller text. At the bottom, display '[AUTHOR/DATE]'.
Use a [COLOR_SCHEME] color scheme with a [STYLE] design. Clean, modern typography.
16:9 widescreen layout. No stock photos — use abstract shapes or gradients for
the background.
```

### Bullet Point Slide

```
Create a presentation slide with the header '[SECTION TITLE]' at the top in bold.
Below the header, display these bullet points clearly:
[BULLET 1]
[BULLET 2]
[BULLET 3]
[BULLET 4]
Each bullet point should be on its own line with clear spacing. Use a [COLOR_SCHEME]
background. Text should be large enough to read from a distance. Professional,
corporate layout. 16:9 widescreen.
```

### Comparison Slide (Two Columns)

```
Create a presentation slide comparing two items side by side. The slide title is
'[TITLE]'. Left column header: '[ITEM A]'. Right column header: '[ITEM B]'.
Left column points: [A_POINTS]. Right column points: [B_POINTS].
Use a clear visual divider between columns. [COLOR_SCHEME] color scheme.
Professional layout with balanced spacing. 16:9 widescreen.
```

### Timeline Infographic

```
Create a horizontal timeline infographic showing these events in chronological order:
[EVENT 1: DATE - DESCRIPTION]
[EVENT 2: DATE - DESCRIPTION]
[EVENT 3: DATE - DESCRIPTION]
[EVENT 4: DATE - DESCRIPTION]
Each event should have a marker/node on the timeline with the date above and
description below. Use [COLOR_SCHEME] colors. Clean, modern design with clear
connecting lines. Title: '[TIMELINE TITLE]'. 16:9 widescreen layout.
```

### Process Flow / Steps Infographic

```
Create an infographic showing a [NUMBER]-step process for '[PROCESS NAME]'.
Display the steps as connected nodes flowing left to right (or top to bottom):
Step 1: [STEP_1]
Step 2: [STEP_2]
Step 3: [STEP_3]
Step 4: [STEP_4]
Use arrows or connectors between steps. Each step should have an icon or visual
marker. [COLOR_SCHEME] color scheme. Clean, professional design. 16:9 layout.
```

### Data Visualization / Stats Slide

```
Create an infographic slide displaying these statistics:
[STAT 1: LABEL - VALUE]
[STAT 2: LABEL - VALUE]
[STAT 3: LABEL - VALUE]
Show each statistic as a large, prominent number with its label below.
Use [VISUAL_TYPE: bar chart / pie chart / icon array / progress circles] to
visualize the data. Title: '[SLIDE TITLE]'. [COLOR_SCHEME] color scheme.
Modern, data-focused design. 16:9 widescreen.
```

### Section Divider Slide

```
Create a section divider slide for a presentation. Display the section title
'[SECTION TITLE]' as large, bold, centered text. Optionally include a brief
tagline: '[TAGLINE]'. Use a [COLOR_SCHEME] background that is visually distinct
from content slides (e.g., darker or accent color). Minimal design. 16:9.
```

### Quote/Highlight Slide

```
Create a presentation slide featuring this quote: "[QUOTE TEXT]" — [ATTRIBUTION].
Display the quote in large, elegant typography with quotation marks.
The attribution should be smaller, right-aligned below the quote.
[COLOR_SCHEME] background. Clean, impactful design. 16:9 widescreen.
```

### Concept Map / Diagram

```
Create a concept map diagram showing the relationships between these concepts:
Central concept: [CENTRAL TOPIC]
Connected concepts: [CONCEPT_1], [CONCEPT_2], [CONCEPT_3], [CONCEPT_4]
Relationships: [CENTRAL] connects to [CONCEPT_1] via '[RELATIONSHIP_LABEL]',
[CENTRAL] connects to [CONCEPT_2] via '[RELATIONSHIP_LABEL]', etc.
Use circles or rounded rectangles for each concept with labeled connecting lines.
[COLOR_SCHEME] colors. Clean, educational style. 16:9 layout.
```

## Multi-Slide Deck Generation

To generate a complete presentation deck, iterate through slides one at a time. Each slide is a separate API call.

### Workflow

1. **Plan the deck**: Based on the user's topic, create an outline of slides (title, content slides, section dividers, conclusion).
2. **Determine a consistent style**: Choose a color scheme, font style, and layout direction that will be applied to ALL slides for visual consistency.
3. **Generate slides sequentially**: For each slide in the outline, construct a prompt using the appropriate template from above, ensuring the style directive is consistent.
4. **Name files sequentially**: Save as `slide_01_title.png`, `slide_02_agenda.png`, `slide_03_content.png`, etc.
5. **Present results**: Show the user each generated slide and offer iteration.

### Style Consistency Directive

Prepend this to EVERY slide prompt in a deck to maintain visual consistency:

```
[STYLE DIRECTIVE - Apply to all slides in this deck]
Color scheme: [PRIMARY_COLOR] primary, [SECONDARY_COLOR] secondary, [ACCENT_COLOR] accent.
Typography: [FONT_STYLE] (e.g., clean sans-serif, modern geometric, etc.).
Background: [BACKGROUND_STYLE] (e.g., dark gradient, light with subtle texture, solid white).
Layout: Professional, balanced, with consistent margins.
[END STYLE DIRECTIVE]
```

### Example: Generating a 3-Slide Deck

```bash
# Define consistent style for the whole deck
STYLE="Color scheme: deep navy blue primary, white text, gold accent. Typography: clean, modern sans-serif. Background: dark navy gradient. Professional, consistent margins."

# Slide 1: Title
PROMPT_1="$STYLE Create a title slide. Title: 'The Future of AI'. Subtitle: 'A Strategic Overview'. Author: 'Jane Smith, CTO'. Date: 'February 2026'. 16:9 widescreen."

# Slide 2: Content
PROMPT_2="$STYLE Create a bullet point slide. Header: 'Key Trends'. Bullets: '1. Generative AI adoption in enterprise', '2. Edge computing meets AI', '3. Responsible AI frameworks', '4. Multimodal foundation models'. 16:9 widescreen."

# Slide 3: Conclusion
PROMPT_3="$STYLE Create a conclusion slide. Large text: 'Questions?'. Below: 'jane@company.com | @janesmith'. Minimal design. 16:9 widescreen."

# Generate each slide
for i in 1 2 3; do
  PROMPT_VAR="PROMPT_$i"
  PROMPT="${!PROMPT_VAR}"

  python3 -c "
import json
data = {
    'contents': [{'parts': [{'text': '''$PROMPT'''}]}],
    'generationConfig': {
        'responseModalities': ['TEXT', 'IMAGE'],
        'imageConfig': {'aspectRatio': '16:9', 'imageSize': '2K'}
    }
}
with open('/tmp/gemini_slide_request.json', 'w') as f:
    json.dump(data, f)
"

  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d @/tmp/gemini_slide_request.json > /tmp/gemini_slide_response.json

  python3 -c "
import json, base64
with open('/tmp/gemini_slide_response.json') as f:
    data = json.load(f)
for part in data.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        with open('slide_0${i}.png', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print('Saved: slide_0${i}.png')
"
  echo "Generated slide $i of 3"
done
```

## Iterating on Existing Slides

When the user wants to modify a generated slide, use the image editing workflow.

### Step 1: Read and encode the existing slide

```bash
SLIDE_PATH="slide_01.png"

# Detect mime type
if [[ "$SLIDE_PATH" == *.png ]]; then
    MIME_TYPE="image/png"
elif [[ "$SLIDE_PATH" == *.jpg ]] || [[ "$SLIDE_PATH" == *.jpeg ]]; then
    MIME_TYPE="image/jpeg"
elif [[ "$SLIDE_PATH" == *.webp ]]; then
    MIME_TYPE="image/webp"
else
    MIME_TYPE="image/png"
fi

# Encode to base64 (works on both macOS and Linux)
if [[ "$(uname)" == "Darwin" ]]; then
    IMG_BASE64=$(base64 -i "$SLIDE_PATH")
else
    IMG_BASE64=$(base64 -w0 "$SLIDE_PATH")
fi
```

### Step 2: Send edit request with the slide image

**IMPORTANT:** Always use a file-based approach for the request body. Base64-encoded images are too large for command-line arguments and will cause "argument list too long" errors.

```bash
EDIT_PROMPT="Modify this presentation slide: change the background color from blue to dark green, keep all text and layout the same."

# Write request to a JSON file (avoids command line length limits)
cat > /tmp/gemini_slide_request.json << JSONEOF
{
  "contents": [{
    "parts": [
      {"text": "$EDIT_PROMPT"},
      {
        "inline_data": {
          "mime_type": "$MIME_TYPE",
          "data": "$IMG_BASE64"
        }
      }
    ]
  }],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageConfig": {
      "aspectRatio": "16:9",
      "imageSize": "2K"
    }
  }
}
JSONEOF

# Call the API using the file
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/gemini_slide_request.json > /tmp/gemini_slide_response.json
```

### Step 3: Save the updated slide

```bash
python3 -c "
import json, base64
with open('/tmp/gemini_slide_response.json') as f:
    data = json.load(f)
for part in data.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        with open('slide_01_v2.png', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print('Saved: slide_01_v2.png')
    elif 'text' in part:
        print(part['text'])
"
```

### Complete Slide Edit Example (File-Based)

```bash
# Variables
SLIDE_PATH="slide_01.png"
EDIT_PROMPT="Change the title to 'Updated Results' and add a fourth bullet point: 'Customer Satisfaction: 95%'"
OUTPUT_PATH="slide_01_edited.png"

# Detect mime type and encode
MIME_TYPE=$([[ "$SLIDE_PATH" == *.png ]] && echo "image/png" || echo "image/jpeg")
IMG_BASE64=$(base64 -i "$SLIDE_PATH" 2>/dev/null || base64 -w0 "$SLIDE_PATH")

# Write request to file (required - base64 images are too large for command line)
cat > /tmp/gemini_slide_request.json << JSONEOF
{
  "contents": [{
    "parts": [
      {"text": "$EDIT_PROMPT"},
      {"inline_data": {"mime_type": "$MIME_TYPE", "data": "$IMG_BASE64"}}
    ]
  }],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageConfig": {
      "aspectRatio": "16:9",
      "imageSize": "2K"
    }
  }
}
JSONEOF

# Call API and extract image
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/gemini_slide_request.json > /tmp/gemini_slide_response.json

# Save the output image
python3 -c "
import json, base64
with open('/tmp/gemini_slide_response.json') as f:
    data = json.load(f)
for part in data.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        with open('$OUTPUT_PATH', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print('Saved: $OUTPUT_PATH')
"
```

## API Usage

### Python SDK: Generate a Slide

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

### Python SDK: Generate a Multi-Slide Deck

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

### JavaScript SDK: Generate a Slide

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

### REST API (curl) Quick Reference

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

Note: For longer prompts or prompts with special characters, always use the file-based approach (write JSON to `/tmp/` and use `curl -d @/tmp/file.json`).

## Prompting Best Practices for Slides & Infographics

### 1. Always Specify the Slide Type
Start every prompt by stating what kind of visual you want: "Create a title slide...", "Create a bullet point slide...", "Create a comparison infographic...". This gives the model the structural context it needs.

### 2. Provide Exact Text Content
The model renders text most accurately when you provide the exact words. Instead of "a slide about quarterly results", write "a slide with the title 'Q4 2025 Results' and the numbers: Revenue $4.2M, Users 50K, Growth 22%".

### 3. Specify Layout Direction
Describe spatial arrangement: "title centered at the top", "three columns below the header", "timeline flowing left to right", "four quadrants".

### 4. Include Color and Style Direction
- Name specific colors: "dark navy blue background with white text and gold accents"
- Reference design styles: "modern corporate", "minimalist", "bold startup pitch deck"
- Specify contrast: "high contrast for readability on projectors"

### 5. Enforce Consistency Across Decks
When generating multiple slides, prepend a style directive to every prompt (see Multi-Slide Deck Generation section). This ensures matching color schemes, fonts, and layout patterns across all slides.

### 6. Text Rendering Tips
- Keep text concise: fewer words render more clearly
- Specify font characteristics descriptively: "large bold sans-serif", "clean monospace"
- For bullet points, number them in the prompt to help the model maintain order
- Request "large enough to read from a distance" for presentation slides

### 7. For Data Visualizations
- Specify the chart type explicitly: "bar chart", "pie chart", "line graph"
- Provide exact data values, not descriptions
- Limit to 5-7 data points per visual for clarity
- Request labels on all data elements

### 8. Aspect Ratio Guidance
- Use 16:9 for all standard presentation slides
- Use 9:16 for vertical infographics (social media stories, scrollable web content)
- Use 1:1 for social media post cards
- Use 21:9 for ultra-wide timelines or panoramic infographics

## Resolution and Aspect Ratio Reference

| Aspect Ratio | 1K Resolution | 2K Resolution | 4K Resolution | Best For              |
|--------------|---------------|---------------|---------------|-----------------------|
| 16:9         | 1376x768      | 2752x1536     | 5504x3072     | Slides (default)      |
| 9:16         | 768x1376      | 1536x2752     | 3072x5504     | Vertical infographics |
| 1:1          | 1024x1024     | 2048x2048     | 4096x4096     | Social media cards    |
| 21:9         | —             | —             | —             | Ultra-wide timelines  |
| 3:2          | 1264x848      | 2528x1696     | 5056x3392     | Wide infographics     |
| 2:3          | 848x1264      | 1696x2528     | 3392x5056     | Tall infographics     |
| 4:3          | —             | —             | —             | Classic slide format   |

**Recommendation**: Use 16:9 at 2K for all presentation slides. This produces 2752x1536 images, which are sharp on all modern displays and projectors.

## Error Handling

Common issues and solutions:

- **No image returned**: Ensure `responseModalities` includes `"IMAGE"` in the `generationConfig`. Text-only responses mean the model chose not to generate an image.
- **Text rendering issues**: If text appears garbled, simplify the prompt. Use fewer words on the slide and spell out exact text to render.
- **Inconsistent styles across deck**: Ensure the style directive is prepended to every prompt in the deck, not just the first one.
- **Safety filters**: Some prompts may be blocked. Rephrase to avoid sensitive content.
- **Rate limits**: Implement exponential backoff. When generating multi-slide decks, add a 1-2 second delay between API calls.
- **"Argument list too long"**: Use the file-based approach (write JSON to `/tmp/` and use `curl -d @/tmp/file.json`).
- **Large output files**: 4K images are large. Use 2K unless print quality is needed.

### Error Check Script

```bash
python3 -c "
import json
with open('/tmp/gemini_slide_response.json') as f:
    data = json.load(f)

if 'error' in data:
    print(f'API Error: {data[\"error\"][\"message\"]}')
    print(f'Code: {data[\"error\"][\"code\"]}')
elif 'candidates' not in data or len(data['candidates']) == 0:
    print('No candidates returned. The prompt may have been filtered.')
elif data['candidates'][0].get('finishReason') == 'SAFETY':
    print('Response blocked by safety filters. Try rephrasing the prompt.')
else:
    has_image = any('inlineData' in p for p in data['candidates'][0]['content']['parts'])
    if not has_image:
        print('No image generated. The model returned text only.')
    else:
        print('Success: Image generated.')
"
```

## Dependencies

For Python SDK usage:
```bash
pip install google-genai pillow
```

For JavaScript SDK usage:
```bash
npm install @google/genai
```

For curl-based workflow (default):
- `curl` (pre-installed on macOS and Linux)
- `python3` (for JSON handling and base64 decoding)
- `base64` (pre-installed on macOS and Linux)

## Important Notes

- All generated images include a SynthID watermark
- The model uses a "thinking" process for complex slide compositions
- For best text rendering, keep text concise and specify exact content
- Generated slide images are not stored by the API — save outputs locally
- Each slide in a deck is a separate API call (no batch generation)
- For maximum text clarity, generate at 2K or higher resolution
- The model works best with English text; other languages are supported but may have reduced text rendering quality
