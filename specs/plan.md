# Langy Prototype Plan

## Overview
Build a single-file HTML prototype in `public/index.html` for testing multiple UI design approaches for the language learning app. Uses Vue.js (CDN), TailwindCSS (CDN), and minimal JavaScript. All user settings and feedback persist via localStorage.

**Design Philosophy:** Learn words in context through full sentences. Don't give away answers immediately. Users should interact to reveal translations.

## Files

1. **public/index.html** - Main prototype file (single-file app)
   - Vue.js 3 (CDN)
   - TailwindCSS (CDN)
   - All JavaScript inline
   - 15 Chinese vocabulary words with sentences

2. **specs/plan.md** - This document
3. **feedback/** - User feedback exports (JSON)

## Features

### Core Functionality
- Display Chinese words **in full sentence context** (mock data)
- Show pinyin and English translation on demand
- **4-level granular response system:**
  - "Know Whole Sentence" - full comprehension (3 points)
  - "Know the Word" - knows target word but not full sentence (2 points)
  - "Uncertain" - partial/unclear knowledge (1 point)
  - "Don't Know" - no knowledge (0 points)
- Next word progression with loop
- Mobile-responsive design
- **localStorage persistence** for all user state
- **High-contrast colors** for accessibility

### Design Variants (5 Options)

Based on user feedback, all designs emphasize:
- Learning in context (full sentences)
- Interactive reveal (don't give away answers)
- Clear visual hierarchy

1. **Card Flip** - Sentence with highlighted word on front, translation on back
   - Front: Full sentence with target word highlighted in amber
   - Shows pinyin for target word
   - Click to flip and see translation

2. **Sentence Highlight** - Click highlighted word to reveal meaning
   - Single sentence with highlighted target word
   - Click word directly to reveal translation
   - Clean, minimal interface

3. **Sentence First** - Read sentence, get help if needed
   - Shows full sentence immediately
   - "Need Help?" button reveals word breakdown
   - Encourages trying to understand before revealing

4. **Split View** - Chinese visible, English locked
   - Left: Chinese word + sentence (always visible)
   - Right: Click to reveal English translation
   - Lock icon indicates hidden content

5. **Minimalist Flow** - One element at a time
   - Step-through presentation
   - Maximum focus on one piece of information
   - Vertical button layout

### Design Feedback Toolbar (Floating, Draggable)
- **Blue header** - drag to move anywhere on screen
- Design switcher dropdown (1-5)
- Feedback text area for current design
- Progress stats (word position, words reviewed)
- Export all feedback button (downloads JSON with timestamp)
- Reset all data button
- Collapse/expand toggle
- Position persists via localStorage

### Backend Debug Panel (Floating, Draggable)
- **Purple header** - drag to move anywhere on screen
- **User Level Estimate** - 0-100% progress bar with confidence level
- **Estimated Vocabulary** - calculated from responses
- **Performance Breakdown:**
  - Know Sentence: count + percentage (3 pts each)
  - Know Word: count + percentage (2 pts each)
  - Uncertain: count + percentage (1 pt each)
  - Don't Know: count + percentage (0 pts each)
- **Simulated Server State** - JSON preview of backend data
- Collapse/expand toggle
- Position persists via localStorage

## Technical Approach

### localStorage Schema
```javascript
{
  currentDesign: 1-5,                    // Selected design variant
  currentWordIndex: 0,                   // Progress through mock words
  designFeedback: {                      // Feedback per design
    "1": "text feedback...",
    "2": "text feedback...",
    // ... etc
  },
  toolbarCollapsed: false,               // Toolbar UI state
  debugPanelCollapsed: false,            // Debug panel UI state
  toolbarPosition: {x: null, y: null},   // Draggable position (null = default)
  debugPosition: {x: null, y: null},     // Debug panel position
  wordResponses: {                       // User responses per word
    "0": "know-sentence",
    "1": "know-word",
    "2": "uncertain",
    "3": "dont-know",
    // ... etc
  },
  sessionData: {
    wordsReviewed: 0,
    lastVisit: timestamp
  },
  backendState: {                        // Computed (saved for export)
    level: 67,                           // 0-100
    estimatedVocab: 450,
    confidence: "High"
  }
}
```

### Data Structure
```javascript
// Mock vocabulary data
{
  word: "学习",
  pinyin: "xuéxí",
  english: "to study",
  sentence: "我每天都在学习中文",
  sentenceEnglish: "I study Chinese every day"
}
```

### State Management
- Vue reactive state synced with localStorage
- Auto-save on state changes
- Load from localStorage on mount
- Graceful fallback if localStorage unavailable

### Level Estimation Algorithm - Bayesian Approach

**Core Problem:** We're estimating a user's unknown true proficiency level through statistical sampling. Each response is a noisy signal about their true level, and we need to:
1. Provide a point estimate (most likely level)
2. Quantify our uncertainty (confidence interval)
3. Update beliefs as we get more data (Bayesian updating)

**Statistical Model: Beta Distribution**

We use a **Beta(α, β) distribution** to model the posterior probability of the user's proficiency:

**Why Beta Distribution?**
- Natural for modeling proportions (0-100%)
- Conjugate prior makes Bayesian updating simple
- Built-in uncertainty that decreases with sample size
- Always produces valid probabilities (bounded [0,1])

**Prior:** Beta(1, 1) = Uniform distribution (no initial bias)

**Weight Mapping:**
- "Know Whole Sentence" → weight = 1.0 (full success)
- "Know the Word" → weight = 0.67 (partial success)
- "Uncertain" → weight = 0.33 (weak signal)
- "Don't Know" → weight = 0.0 (failure)

**Posterior Update:**
For each response with weight w:
```
α_new = α_old + w
β_new = β_old + (1 - w)
```

**Point Estimate (Mean):**
```
level = α / (α + β) × 100
```

**Uncertainty (Standard Deviation):**
```
variance = (α × β) / [(α + β)² × (α + β + 1)]
stdDev = √variance × 100
```

**95% Credible Interval:**
```
marginOfError = 1.96 × stdDev
CI_lower = max(0, level - marginOfError)
CI_upper = min(100, level + marginOfError)
```

**Example Progression:**
```
n=1,  "Know Sentence" → 67% ± 27% (CI: 40%-94%)   Confidence: Very Low
n=2,  mixed responses → 62% ± 21% (CI: 41%-83%)   Confidence: Very Low
n=5,  mixed responses → 58% ± 14% (CI: 44%-72%)   Confidence: Medium
n=10, mixed responses → 57% ± 10% (CI: 47%-67%)   Confidence: High
n=20, mixed responses → 56% ± 7%  (CI: 49%-63%)   Confidence: Very High
```

**Confidence Levels (based on uncertainty, not just sample size):**
- Very Low: < 3 responses
- Low: stdDev > 20%
- Medium: stdDev > 12%
- High: stdDev > 7%
- Very High: stdDev ≤ 7%

**Vocabulary Estimation:**
```
estimatedVocab = (knowSentence × 150) + (knowWord × 100) + (uncertain × 50)
```

**Visualization:**
- **Light purple band** = 95% confidence interval (uncertainty range)
- **Dark purple bar** = point estimate (most likely level)
- Band gets narrower as user answers more questions
- Text shows: "X% ± Y%, CI: [A%-B%], Confidence: Level"

### Styling Strategy
- TailwindCSS for all layout and utilities
- High-contrast colors (darker shades: 600-800 instead of 500)
- Amber instead of yellow for better visibility
- Custom CSS for flip card animation
- Mobile-first responsive with flex-wrap for buttons
- No dark mode (prototype only)

## Implementation History

✅ Completed:
1. Basic HTML structure with Vue 3 app
2. TailwindCSS and Vue.js CDN integration
3. localStorage utilities (load/save/clear/reset)
4. 15 realistic Chinese vocabulary words with sentences
5. Draggable floating toolbar (blue header)
6. Draggable backend debug panel (purple header)
7. 5 design variants based on user feedback
8. 4-level granular response system
9. Design switcher with persistence
10. Feedback capture with auto-save
11. Export functionality (JSON with timestamp)
12. Backend level estimation algorithm
13. High-contrast color improvements
14. Card flip layout fixes
15. Responsive design with flex-wrap buttons

## Key Design Decisions

### Based on User Feedback:
- **Sentence-first approach**: All designs show full sentences (learning in context)
- **Interactive reveal**: Don't give away answers; require user interaction
- **4-level responses**: Distinguish between knowing word vs. knowing whole sentence
- **Removed "Progressive Reveal"**: Replaced with "Sentence First" design
- **Split View hides English**: Shows lock icon, click to reveal
- **Sentence Highlight simplified**: Single sentence, click word directly

### Technical Decisions:
- **localStorage only**: No backend (prototype phase)
- **Single HTML file**: Easier to test and share
- **Draggable panels**: Better UX for different screen sizes
- **JSON export**: Includes all feedback + backend state snapshot
- **High contrast**: Accessibility (darker colors: 600-800)
- **Amber highlights**: Better visibility than yellow

## Known Issues & Lessons Learned

### Issue: Word Not Found in Sentence (Fixed)
**Problem:** The word "时间" was set with sentence "现在几点了？" but "时间" doesn't appear in that sentence. This caused the highlighting logic to fail because `indexOf()` returned -1.

**Root Cause:** Mock data was created by writing natural-sounding sentences without programmatically verifying that the target word actually appears in each sentence.

**Impact:**
- Highlighting showed wrong characters
- User saw incorrect word being taught
- Data integrity issue

**Fix:** Changed sentence to "你有时间吗？" which contains "时间"

**Prevention Strategy:**
1. **Validation Function Needed:** Add a computed property or validation that checks:
   ```javascript
   sentence.indexOf(word) !== -1
   ```
2. **Data Quality Check:** Before deploying vocabulary:
   - Manually verify each word appears in its sentence
   - Or use automated test to validate all vocabulary entries
3. **Error Handling:** Add fallback behavior when `indexOf` returns -1:
   - Could show error in debug panel
   - Could skip highlighting if word not found
   - Could log warning to console

**Future Enhancement:** Add validation to `mounted()` hook:
```javascript
mounted() {
    // Validate vocabulary data
    this.vocabulary.forEach((item, idx) => {
        if (item.sentence.indexOf(item.word) === -1) {
            console.warn(`Word "${item.word}" not found in sentence at index ${idx}`);
        }
    });
    this.loadFromLocalStorage();
}
```

## Security & Environment Configuration

### Overview
When transitioning from prototype to production with a server component, proper secrets management is critical. This section outlines security best practices for the eventual backend.

### Environment Variables (.env)

**Purpose:**
- Store sensitive credentials outside of source code
- Different configurations for development, staging, production
- Never commit secrets to git

**Required Environment Variables:**

```bash
# .env file (NEVER commit to git!)

# OpenAI API (for sentence generation)
OPENAI_API_KEY=sk-...your-key-here...

# Database connection
DATABASE_URL=postgresql://user:password@host:port/dbname
DATABASE_SSL=true

# Session/Authentication
SESSION_SECRET=random-64-char-hex-string
JWT_SECRET=another-random-64-char-hex-string

# Application
NODE_ENV=development  # or production
PORT=3000
FRONTEND_URL=http://localhost:8080

# Optional: Chinese NLP services
JIEBA_MODEL_PATH=/path/to/jieba/models
PINYIN_API_KEY=...if-using-external-service...

# Analytics (optional)
ANALYTICS_KEY=...
```

**Example .env.example (safe to commit):**

```bash
# Copy this to .env and fill in your actual values
OPENAI_API_KEY=your_openai_api_key_here
DATABASE_URL=your_database_connection_string
SESSION_SECRET=generate_random_secret_here
JWT_SECRET=generate_another_random_secret_here
NODE_ENV=development
PORT=3000
```

### .gitignore Configuration

**Critical files to exclude:**

```gitignore
# Environment variables
.env
.env.local
.env.*.local

# Credentials
credentials.json
service-account-key.json
*.pem
*.key

# Secrets
secrets/
config/secrets.yml

# Database
*.sqlite
*.db
database.json

# Logs (may contain sensitive data)
logs/
*.log

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp

# Dependencies
node_modules/
venv/
__pycache__/

# Build artifacts
dist/
build/
*.pyc
```

### Server Architecture Security

**Backend Stack (Python recommended):**

```python
# server.py example structure
from flask import Flask
from dotenv import load_dotenv
import os

# Load environment variables
load_dotenv()

app = Flask(__name__)

# Access secrets safely
OPENAI_KEY = os.getenv('OPENAI_API_KEY')
DATABASE_URL = os.getenv('DATABASE_URL')

# NEVER do this:
# OPENAI_KEY = "sk-hardcoded-key"  ❌ BAD!

# Configuration class
class Config:
    SECRET_KEY = os.environ.get('SESSION_SECRET')
    DATABASE_URI = os.environ.get('DATABASE_URL')
    OPENAI_API_KEY = os.environ.get('OPENAI_API_KEY')

    @staticmethod
    def validate():
        """Fail fast if required env vars missing"""
        required = ['SESSION_SECRET', 'DATABASE_URL', 'OPENAI_API_KEY']
        missing = [var for var in required if not os.getenv(var)]
        if missing:
            raise ValueError(f"Missing required env vars: {missing}")

# Validate on startup
Config.validate()
```

### Security Best Practices

**1. API Key Management:**
- ✅ Store in environment variables
- ✅ Rotate keys periodically
- ✅ Use different keys for dev/staging/prod
- ❌ Never commit to git
- ❌ Never log API keys
- ❌ Never send to frontend

**2. Database Security:**
- Use connection pooling
- Enable SSL/TLS for connections
- Use parameterized queries (prevent SQL injection)
- Implement row-level security for user data
- Regular backups with encryption

**3. User Data Protection:**
- Hash passwords (bcrypt, argon2)
- Encrypt sensitive user data at rest
- Use HTTPS only (no HTTP)
- Implement CSRF protection
- Rate limiting on API endpoints
- Input validation and sanitization

**4. Session Management:**
- Secure, httpOnly cookies
- Short session expiry
- Regenerate session IDs after login
- Implement logout functionality
- Monitor for suspicious activity

### Deployment Checklist

Before deploying to production:

- [ ] All secrets in environment variables (not code)
- [ ] .env files in .gitignore
- [ ] HTTPS enabled with valid certificate
- [ ] Database connections use SSL
- [ ] API rate limiting implemented
- [ ] Input validation on all endpoints
- [ ] Error messages don't leak sensitive info
- [ ] Logging configured (without secrets)
- [ ] Security headers set (CSP, HSTS, etc.)
- [ ] Dependencies scanned for vulnerabilities
- [ ] Backup strategy implemented
- [ ] Monitoring and alerting set up

### Local Development Setup

**For new developers:**

```bash
# 1. Clone repository
git clone https://github.com/kazad/langy.git
cd langy

# 2. Copy example env file
cp .env.example .env

# 3. Edit .env with your credentials
nano .env  # or use your editor

# 4. Never commit .env
git status  # should NOT show .env

# 5. Install dependencies
pip install -r requirements.txt  # Python
# or
npm install  # Node.js

# 6. Verify configuration
python -c "from dotenv import load_dotenv; load_dotenv(); import os; print('✓ Config loaded')"
```

### Production Deployment

**Environment variables on hosting platforms:**

**Heroku:**
```bash
heroku config:set OPENAI_API_KEY=sk-...
heroku config:set DATABASE_URL=postgresql://...
```

**Vercel:**
```bash
vercel env add OPENAI_API_KEY
vercel env add DATABASE_URL
```

**Docker:**
```yaml
# docker-compose.yml
services:
  web:
    env_file:
      - .env
    environment:
      - NODE_ENV=production
```

**AWS / Cloud:**
- Use AWS Secrets Manager or Parameter Store
- Azure Key Vault
- Google Cloud Secret Manager

### Monitoring & Alerts

**What to monitor:**
- Failed login attempts (potential attacks)
- API key usage (detect leaks)
- Database connection errors
- Unusual traffic patterns
- Error rates by endpoint

**Alert on:**
- API key used from unexpected IP
- Sudden spike in requests
- Database connection failures
- Authentication errors above threshold

## Future Enhancements
- Connect to real backend API (with proper .env configuration)
- Implement actual Bayesian model server-side
- Add more vocabulary words with frequency ranking
- Support multiple languages
- Track time decay (forgetting curve)
- Audio pronunciation (external API with key in .env)
- Spaced repetition algorithm
- **Add vocabulary data validation** to catch mismatches
- Implement user authentication (OAuth, JWT)
- Rate limiting and abuse prevention
