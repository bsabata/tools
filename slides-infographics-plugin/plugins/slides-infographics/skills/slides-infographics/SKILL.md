---
name: Slides & Infographics
description: This skill should be used when the user wants to create presentation slides, slide decks, infographics, timelines, comparison charts, process flows, concept maps, data visualizations, diagrams, or any visual summary. Common triggers include "create a slide about...", "make a presentation on...", "design an infographic showing...", "build a timeline of...", "generate a slide deck for...", "create a comparison chart", "make a visual summary of...", "create a diagram of...", and "visualize...". Generates images using the Gemini image generation API.
---

# Slides & Infographics

Generate presentation slides, infographics, and visual decks by calling the Gemini image generation API (`gemini-3-pro-image-preview`) via curl and Python.

## Setup Required

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
| Timeline (wide)      | 21:9         | 2K*        | Ultra-wide for horizontal timelines      |
| Poster/Print         | 3:4          | 4K         | Portrait print format                    |
| Comparison Chart     | 16:9         | 2K         | Side-by-side layout                      |

*Exact pixel dimensions for 21:9 are determined automatically by the API. For detailed resolution values per aspect ratio, read `references/resolution-reference.md`.

## Core Workflow: Generate a Single Slide

### Step 1: Construct the slide prompt

Build a detailed prompt that includes:
- **Slide type** (title, bullet points, comparison, timeline, etc.)
- **Content** (exact text, data points, labels)
- **Visual style** (color scheme, layout, typography direction)
- **Aspect ratio** (default 16:9 for slides)

For ready-to-use prompt templates for each slide type, read `references/prompt-templates.md`.

### Step 2: Write the prompt and build the request JSON safely

**IMPORTANT:** Always use a file-based approach with unique temp files. This avoids "argument list too long" errors, safely handles special characters in prompts, and prevents conflicts in concurrent sessions.

```bash
# Create unique temp files
REQUEST_FILE=$(mktemp /tmp/gemini_req_XXXXXX.json)
RESPONSE_FILE=$(mktemp /tmp/gemini_resp_XXXXXX.json)
PROMPT_FILE=$(mktemp /tmp/gemini_prompt_XXXXXX.txt)

# Write the prompt to a file (quoted delimiter prevents shell expansion)
cat > "$PROMPT_FILE" << 'PROMPT_EOF'
Create a professional presentation slide with the title 'AI in Healthcare' as
large bold text centered at the top. Below the title, show three bullet points:
'1. Diagnostic Accuracy', '2. Drug Discovery', '3. Patient Monitoring'. Use a
modern dark blue gradient background with white text. Clean, corporate design.
16:9 widescreen layout.
PROMPT_EOF

# Use Python to safely construct JSON from the prompt file
python3 -c "
import json, sys
with open(sys.argv[1]) as f:
    prompt_text = f.read().strip()
data = {
    'contents': [{'parts': [{'text': prompt_text}]}],
    'generationConfig': {
        'responseModalities': ['TEXT', 'IMAGE'],
        'imageConfig': {
            'aspectRatio': '16:9',
            'imageSize': '2K'
        }
    }
}
with open(sys.argv[2], 'w') as f:
    json.dump(data, f)
" "$PROMPT_FILE" "$REQUEST_FILE"
```

### Step 3: Call the API

```bash
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$REQUEST_FILE" > "$RESPONSE_FILE"
```

### Step 4: Extract and save the slide image

```bash
python3 -c "
import json, base64, sys
with open(sys.argv[1]) as f:
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
" "$RESPONSE_FILE"
```

## Multi-Slide Deck Generation

To generate a complete presentation deck, iterate through slides one at a time. Each slide is a separate API call.

### Workflow

1. **Plan the deck**: Based on the user's topic, create an outline of slides (title, content slides, section dividers, conclusion).
2. **Determine a consistent style**: Choose a color scheme, font style, and layout direction that will be applied to ALL slides.
3. **Generate slides sequentially**: For each slide, construct a prompt using the appropriate template from `references/prompt-templates.md`, prepending the style directive.
4. **Name files sequentially**: Save as `slide_01_title.png`, `slide_02_agenda.png`, etc.
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
STYLE="Color scheme: deep navy blue primary, white text, gold accent. Typography: clean, modern sans-serif. Background: dark navy gradient. Professional, consistent margins."

PROMPTS=(
  "$STYLE Create a title slide. Title: 'The Future of AI'. Subtitle: 'A Strategic Overview'. Author: 'Jane Smith, CTO'. Date: 'February 2026'. 16:9 widescreen."
  "$STYLE Create a bullet point slide. Header: 'Key Trends'. Bullets: '1. Generative AI adoption in enterprise', '2. Edge computing meets AI', '3. Responsible AI frameworks', '4. Multimodal foundation models'. 16:9 widescreen."
  "$STYLE Create a conclusion slide. Large text: 'Questions?'. Below: 'jane@company.com | @janesmith'. Minimal design. 16:9 widescreen."
)

for i in "${!PROMPTS[@]}"; do
  SLIDE_NUM=$((i + 1))
  REQUEST_FILE=$(mktemp /tmp/gemini_req_XXXXXX.json)
  RESPONSE_FILE=$(mktemp /tmp/gemini_resp_XXXXXX.json)
  PROMPT_FILE=$(mktemp /tmp/gemini_prompt_XXXXXX.txt)

  # Write prompt safely to file
  printf '%s' "${PROMPTS[$i]}" > "$PROMPT_FILE"

  python3 -c "
import json, sys
with open(sys.argv[1]) as f:
    prompt_text = f.read()
data = {
    'contents': [{'parts': [{'text': prompt_text}]}],
    'generationConfig': {
        'responseModalities': ['TEXT', 'IMAGE'],
        'imageConfig': {'aspectRatio': '16:9', 'imageSize': '2K'}
    }
}
with open(sys.argv[2], 'w') as f:
    json.dump(data, f)
" "$PROMPT_FILE" "$REQUEST_FILE"

  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d @"$REQUEST_FILE" > "$RESPONSE_FILE"

  python3 -c "
import json, base64, sys
with open(sys.argv[1]) as f:
    data = json.load(f)
for part in data.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        filename = f'slide_{int(sys.argv[2]):02d}.png'
        with open(filename, 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print(f'Saved: {filename}')
" "$RESPONSE_FILE" "$SLIDE_NUM"

  echo "Generated slide $SLIDE_NUM of ${#PROMPTS[@]}"
done
```

## Iterating on Existing Slides

When the user wants to modify a generated slide, use the image editing workflow. Python handles file reading, base64 encoding, and JSON construction safely.

```bash
SLIDE_PATH="slide_01.png"
OUTPUT_PATH="slide_01_v2.png"
REQUEST_FILE=$(mktemp /tmp/gemini_req_XXXXXX.json)
RESPONSE_FILE=$(mktemp /tmp/gemini_resp_XXXXXX.json)
PROMPT_FILE=$(mktemp /tmp/gemini_prompt_XXXXXX.txt)

# Write edit prompt to file (quoted delimiter prevents shell expansion)
cat > "$PROMPT_FILE" << 'PROMPT_EOF'
Modify this presentation slide: change the background color from blue to dark
green, keep all text and layout the same.
PROMPT_EOF

# Detect mime type
MIME_TYPE=$([[ "$SLIDE_PATH" == *.png ]] && echo "image/png" || echo "image/jpeg")

# Python reads the image file directly, encodes it, and builds safe JSON
python3 -c "
import json, sys, base64
with open(sys.argv[1]) as f:
    prompt_text = f.read().strip()
with open(sys.argv[2], 'rb') as f:
    img_base64 = base64.b64encode(f.read()).decode('utf-8')
data = {
    'contents': [{'parts': [
        {'text': prompt_text},
        {'inlineData': {'mimeType': sys.argv[3], 'data': img_base64}}
    ]}],
    'generationConfig': {
        'responseModalities': ['TEXT', 'IMAGE'],
        'imageConfig': {'aspectRatio': '16:9', 'imageSize': '2K'}
    }
}
with open(sys.argv[4], 'w') as f:
    json.dump(data, f)
" "$PROMPT_FILE" "$SLIDE_PATH" "$MIME_TYPE" "$REQUEST_FILE"

# Call the API
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$REQUEST_FILE" > "$RESPONSE_FILE"

# Save the updated slide
python3 -c "
import json, base64, sys
with open(sys.argv[1]) as f:
    data = json.load(f)
for part in data.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        with open(sys.argv[2], 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print(f'Saved: {sys.argv[2]}')
    elif 'text' in part:
        print(part['text'])
" "$RESPONSE_FILE" "$OUTPUT_PATH"
```

## Prompting Best Practices

1. **Always specify the slide type**: Start every prompt with "Create a title slide...", "Create a bullet point slide...", etc.
2. **Provide exact text content**: Instead of "a slide about quarterly results", write "a slide with the title 'Q4 2025 Results' and the numbers: Revenue $4.2M, Users 50K, Growth 22%".
3. **Specify layout direction**: Describe spatial arrangement: "title centered at the top", "three columns below the header", "timeline flowing left to right".
4. **Include color and style direction**: Name specific colors ("dark navy blue background with white text and gold accents") and reference design styles ("modern corporate", "minimalist").
5. **Enforce consistency across decks**: Prepend the style directive to every prompt in a deck, not just the first one.
6. **Keep text concise**: Fewer words render more clearly. Request "large enough to read from a distance" for presentation slides.
7. **For data visualizations**: Specify the chart type explicitly, provide exact data values, and limit to 5-7 data points per visual.
8. **Aspect ratio guidance**: Use 16:9 for slides, 9:16 for vertical infographics, 1:1 for social cards, 21:9 for timelines.

## Dependencies

For the default curl-based workflow:
- `curl` (pre-installed on macOS and Linux)
- `python3` (for JSON handling and base64 decoding)

For SDK alternatives, see `references/sdk-examples.md`.

## Important Notes

- All generated images include a SynthID watermark
- For best text rendering, keep text concise and specify exact content
- Generated slide images are not stored by the API — save outputs locally
- Each slide in a deck is a separate API call (no batch generation)
- Use 2K or higher resolution for maximum text clarity
- English text renders best; other languages are supported but may have reduced quality

For error handling and debugging, read `references/error-handling.md`.
