# Langy - Context-Based Language Learning

A language learning app prototype that emphasizes learning words in context through full sentences, with Bayesian uncertainty estimation for user proficiency levels.

## 🎯 Core Philosophy

- **Learn in Context**: Words presented in full sentences, not isolation
- **Interactive Reveal**: Don't give away answers; require user interaction
- **Statistical Rigor**: Bayesian estimation with proper uncertainty quantification
- **Data-Driven**: 4-level granular responses to better estimate proficiency

## ✨ Features

### Current (Prototype)
- 5 different UI design variants for user testing
- 15 realistic Chinese vocabulary words with example sentences
- **Bayesian level estimation** with 95% confidence intervals
- Visual uncertainty bands that narrow as you answer more questions
- Draggable feedback and debug panels
- localStorage persistence (no backend needed for testing)
- Design feedback collection and JSON export

### Response System (4 Levels)
1. **Know Whole Sentence** (3 pts) - Full comprehension
2. **Know the Word** (2 pts) - Know target word, not full sentence
3. **Uncertain** (1 pt) - Partial/unclear knowledge
4. **Don't Know** (0 pts) - No knowledge

### Statistical Model

Uses **Beta Distribution** for proficiency estimation:
- Starts with complete uncertainty (uniform prior)
- Each response updates the posterior distribution
- Provides point estimate + 95% credible interval
- Uncertainty decreases with more responses

**Example progression:**
```
n=1  → 67% ± 27% (CI: 40%-94%)   Very uncertain!
n=5  → 58% ± 14% (CI: 44%-72%)   Getting clearer
n=10 → 57% ± 10% (CI: 47%-67%)   Moderate confidence
n=20 → 56% ± 7%  (CI: 49%-63%)   High confidence
```

## 🚀 Quick Start

### Option 1: Direct File
Simply open `public/index.html` in your browser. No build step needed!

### Option 2: Local Server
```bash
# Python
python -m http.server 8000
# Then open http://localhost:8000/public/

# Node.js
npx http-server public -p 8000
```

### Try It Out
1. Open the prototype
2. Test the 5 different design variants (switch via dropdown)
3. Leave feedback on each design
4. Watch the Backend Debug panel show your estimated level with uncertainty
5. Export your feedback as JSON

## 📊 Design Variants

1. **Card Flip** - Sentence with highlighted word, click to flip for translation
2. **Sentence Highlight** - Click highlighted word to reveal meaning
3. **Sentence First** - Read sentence, get help on demand
4. **Split View** - Chinese visible, English locked until clicked
5. **Minimalist Flow** - One element at a time, tap through phases

## 🛠️ Tech Stack

**Current (Prototype):**
- Vue.js 3 (CDN)
- TailwindCSS (CDN)
- Single HTML file
- localStorage for persistence

**Planned (Production):**
- Python backend (Flask/FastAPI)
- PostgreSQL database
- OpenAI API for sentence generation
- Jieba for Chinese text segmentation

## 📖 Documentation

See [`specs/plan.md`](specs/plan.md) for:
- Complete technical specification
- Bayesian estimation algorithm details
- Security & environment configuration
- Future enhancement plans
- Implementation history

## 🔒 Security

When building the backend:
- Use `.env` files for all secrets
- Never commit credentials to git
- See detailed security guidelines in `specs/plan.md`

## 🤝 Contributing

This is currently a prototype for design testing. Feedback welcome!

1. Try all 5 design variants
2. Leave feedback via the in-app toolbar
3. Export and share your feedback JSON
4. Open issues with suggestions

## 📝 License

[Add your license here]

## 🙏 Acknowledgments

Built with Claude Code for rapid prototyping and iteration.
