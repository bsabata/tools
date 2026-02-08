# Error Handling

## Common Issues and Solutions

- **No image returned**: Ensure `responseModalities` includes `"IMAGE"` in the `generationConfig`. Text-only responses mean the model chose not to generate an image.
- **Text rendering issues**: If text appears garbled, simplify the prompt. Use fewer words on the slide and spell out exact text to render.
- **Inconsistent styles across deck**: Ensure the style directive is prepended to every prompt in the deck, not just the first one.
- **Safety filters**: Some prompts may be blocked. Rephrase to avoid sensitive content.
- **Rate limits**: Implement exponential backoff. When generating multi-slide decks, add a 1-2 second delay between API calls.
- **"Argument list too long"**: Use the file-based approach (write JSON to a temp file and use `curl -d @file.json`).
- **Large output files**: 4K images are large. Use 2K unless print quality is needed.

## Error Check Script

Use this script to diagnose API response issues:

```bash
python3 -c "
import json, sys

with open(sys.argv[1]) as f:
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
" "$RESPONSE_FILE"
```

Replace `$RESPONSE_FILE` with the actual response file path from your workflow (the `mktemp` output).

## Debugging Tips

1. **Check the raw response**: Read the response JSON file to see exactly what the API returned.
2. **Verify the API key**: Run `echo $GEMINI_API_KEY | head -c 10` to confirm the key is set (shows first 10 chars).
3. **Test with a simple prompt first**: If a complex slide fails, try a minimal prompt like "Create a blue slide with the word 'Test' in white" to isolate the issue.
4. **Check for quota/billing issues**: API errors with code 429 indicate rate limiting; code 403 may indicate billing or quota issues.
