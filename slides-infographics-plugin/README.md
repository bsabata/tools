# Slides & Infographics Plugin

Open-source Claude Code plugin for creating presentation slides, infographics, timelines, and visual decks using Google's Gemini Nano Banana Pro model (`gemini-3-pro-image-preview`).

## Installation

### 1. Add the marketplace to Claude Code

```
/plugin marketplace add <your-github-org>/slides-infographics-plugin
```

### 2. Install the plugin

```
/plugin install slides-infographics@slides-infographics-plugin
```

### 3. Set up your API key

Get a free API key from [Google AI Studio](https://aistudio.google.com/) and export it:

```bash
export GEMINI_API_KEY="your_api_key_here"
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [slides-infographics](./plugins/slides-infographics) | Create slides, infographics, timelines, and presentation decks using Gemini Nano Banana Pro |

## What You Can Create

- **Presentation Slides** - Title slides, bullet points, section headers with professional layouts
- **Infographics** - Data visualizations, process flows, comparison charts
- **Timelines** - Chronological event displays with visual markers
- **Slide Decks** - Complete multi-slide presentations with consistent styling
- **Educational Visuals** - Diagrams, concept maps, explainer graphics
- **Social Media Cards** - Square and vertical format visuals

## Contributing

We welcome contributions! To add a new plugin or improve an existing one:

1. Fork this repository
2. Create your plugin under `plugins/your-plugin-name/`
3. Follow the existing plugin structure (see `plugins/slides-infographics/` for reference)
4. Submit a pull request

## License

Open source - see individual plugins for their specific licenses.
