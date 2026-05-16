---
name: testing-prompt-generator
description: Test the Ferreira.Eletrotech Instagram prompt generator. Use when verifying prompt quality, randomization, deduplication, or UI changes.
---

# Testing the Prompt Generator

## Prerequisites
- Open `index.html` in Chrome (it's a single-file app, no server needed)
- For end-to-end testing: a valid Gemini or Groq API key

## Devin Secrets Needed
- `GEMINI_API_KEY` - Google Gemini API key for testing prompt generation (might expire, check validity first)
- Alternatively, a Groq API key can be configured manually in the Config tab

## Quick Validation (No API Key Needed)
Use Playwright CDP (`http://localhost:29229`) or browser console to call functions directly:

```javascript
// Test prompt text generation
var prompt = buildPromptText('eletrica', 3, 'post_1x1', 'informal', '');
// Verify: contains "fotografo documental brasileiro", no "Canon EOS", no "softbox"

// Test shuffle
shuffleArr([1,2,3,4,5]); // Should produce random order each time

// Test array sizes
console.log(angulos.length, moods.length, detalhesRealismo.length, composicoes.length, horaDia.length);
// Expected: 20, 20, 20, 10, 7

// Test dedup tracking
localStorage.removeItem('fe_recent_temas');
buildPromptText('eletrica', 3, 'post_1x1', 'informal', '');
JSON.parse(localStorage.getItem('fe_recent_temas')); // Should have 3 unique entries

// Test history awareness
getRecentHistory(); // Returns recent titles to append to prompt
```

## localStorage Keys
- `fe_provider` - AI provider ('groq' or 'gemini')
- `fe_apikey` - API key (note: NOT `fe_api_key`)
- `fe_model` - Groq model name
- `fe_recent_temas` - Last 40 used themes for deduplication
- `fe_history` - Generation history
- `fe_theme` - UI theme ('dark' or 'light')
- `fe_saved` - Saved/favorited prompts

## Setting API Key via Playwright
```python
await page.evaluate(f"""
    localStorage.setItem('fe_provider', 'gemini');
    localStorage.setItem('fe_apikey', '{key}');
""")
await page.reload()
```

## End-to-End Testing
1. Configure API key in Config tab
2. Click "Testar Conexao" - should show green "conectado com sucesso!"
3. Go to Gerador tab, select category, format, tone
4. Click GERAR PROMPTS
5. Verify generated cards have: titulo, descricao, prompt (English), legenda (Portuguese)
6. Check prompt doesn't mention camera models or studio lighting
7. Generate again - themes should not repeat (dedup)

## Common Issues
- **HTTP 400 on test connection**: API key might be expired. Test with curl first:
  ```bash
  curl -s "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=$GEMINI_API_KEY" -H "Content-Type: application/json" -d '{"contents":[{"parts":[{"text":"OK"}]}]}'
  ```
- **localStorage key mismatch**: The key is `fe_apikey` (no underscore between api and key)
- **Prompt looks artificial**: Check that buildPromptText output doesn't contain Canon/Nikon/Sony or softbox/key light terms
