# Slides & Infographics Plugin

Create presentation slides and infographics using the Gemini image generation API (`gemini-3-pro-image-preview`).

## Features

- **Presentation Slides**: Title slides, bullet points, section headers with visual layouts
- **Infographics**: Data visualizations, process flows, comparison charts
- **Timelines**: Chronological event displays with visual markers
- **Slide Decks**: Generate multiple sequential slides for complete presentations
- **Educational Visuals**: Diagrams, concept maps, explainer graphics
- **High Resolution**: Default 2K at 16:9 for presentation-ready output
- **Accurate Text Rendering**: Clean, legible text on all slide elements
- **Multiple Aspect Ratios**: 16:9 (slides), 9:16 (stories), 1:1 (social), 21:9 (timelines)

## Setup (Required)

Before using this plugin, you must set your Gemini API key as an environment variable:

1. **Get a free API key** from [Google AI Studio](https://aistudio.google.com/)

2. **Export the key** in your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):
   ```bash
   export GEMINI_API_KEY="your_api_key_here"
   ```

3. **Restart your terminal** or run `source ~/.zshrc` (or `~/.bashrc`)

## Usage

Invoke this skill when you want to:
- "Create a slide about..."
- "Make a presentation on..."
- "Design an infographic showing..."
- "Build a timeline of..."
- "Create a comparison chart..."
- "Generate a slide deck for..."
- "Make a visual summary of..."

## Examples

### Title Slide
```
Create a title slide for a presentation on AI in Healthcare with a modern blue gradient
```

### Bullet Point Slide
```
Create a slide with the header 'Key Benefits' and bullet points: faster processing, lower costs, better accuracy
```

### Infographic
```
Design an infographic comparing renewable energy sources: solar, wind, and hydro
```

### Multi-Slide Deck
```
Generate a 5-slide presentation about machine learning fundamentals
```

### Timeline
```
Create a timeline infographic of major space exploration milestones from 1957 to 2024
```

### Process Flow
```
Create a 4-step process flow infographic for the software development lifecycle
```

## Model Capabilities

The `gemini-3-pro-image-preview` model features:
- Advanced reasoning for complex slide compositions
- "Thinking" process for layout and text rendering refinement
- Up to 14 reference images for input
- Google Search grounding for real-time data
- High-resolution output up to 4K

## Limitations

- Best performance with: EN, ar-EG, de-DE, es-MX, fr-FR, hi-IN, id-ID, it-IT, ja-JP, ko-KR, pt-BR, ru-RU, ua-UA, vi-VN, zh-CN
- No audio or video inputs
- Each slide in a deck is a separate API call
- All generated images include a SynthID watermark
- Text rendering quality improves with concise, explicit text in prompts

## Version

- Version: 1.0.0
- Author: sabata.net
