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

### Level Estimation Algorithm

**Scoring System:**
- Know Whole Sentence: 3 points
- Know the Word: 2 points
- Uncertain: 1 point
- Don't Know: 0 points

**Level Calculation:**
```
level = (totalPoints / maxPossiblePoints) * 100
where maxPossiblePoints = totalResponses * 3
```

**Vocabulary Estimation:**
```
estimatedVocab = (knowSentence * 150) + (knowWord * 100) + (uncertain * 50)
```

**Confidence Levels:**
- Low: < 5 responses
- Medium: 5-9 responses
- High: 10+ responses

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

## Future Enhancements
- Connect to real backend API
- Implement actual Bayesian model for level estimation
- Add more vocabulary words with frequency ranking
- Support multiple languages
- Track time decay (forgetting curve)
- Audio pronunciation
- Spaced repetition algorithm
- **Add vocabulary data validation** to catch mismatches
