# Design Switcher - Universal Implementation Guide

**TL;DR:** A design switcher lets you build 3-5 different UI variants in a single app, collect user feedback on each, and make data-driven design decisions. Perfect for prototyping with AI tools where you can quickly generate multiple approaches but aren't sure which works best until users test them.

---

A comprehensive guide for building an effective design variant testing system for any web application. This pattern is especially useful for AI-assisted development where you can quickly generate multiple design approaches and need user feedback to choose the best one.

## Overview

A design switcher allows you to test multiple UI/UX approaches in a single application, collect user feedback, and make data-driven design decisions. This is especially valuable during prototyping when you haven't settled on a final design direction.

**Perfect for AI-assisted development:** When using AI tools to build applications, you can quickly generate 3-5 different design variants for the same feature. A design switcher lets users test all variants and provide feedback, helping you choose the winning design based on real user input rather than guesswork.

## Why Use a Design Switcher?

### Benefits
- **Rapid A/B testing** without separate deployments
- **User feedback collection** tied to specific designs
- **Side-by-side comparison** by quickly switching
- **Data-driven decisions** with quantitative and qualitative feedback
- **Lower friction** than asking users to test multiple prototypes
- **Context preservation** - user sees all designs with same content

### When to Use
✅ **Good for:**
- Early prototyping phase
- Testing fundamentally different interaction patterns
- Evaluating multiple design directions simultaneously
- User research sessions
- Stakeholder demos
- AI-generated design variants (quick iteration)
- Solo developer wanting user feedback

❌ **Not ideal for:**
- Production applications (use proper A/B testing frameworks like Optimizely, Google Optimize)
- Testing minor variations (use CSS/theming instead)
- Final stages before launch (should have decided on design)
- Statistical significance testing (need proper A/B testing with traffic splitting)

## Core Components

### 1. Design Variant System

**Key Requirements:**
- Each design should be a complete, self-contained implementation
- Clear separation between designs (no shared state that breaks switching)
- Consistent data across all designs (same content/items shown to user)

**Implementation Pattern (Vue.js example):**

```javascript
data() {
    return {
        currentDesign: 1,  // Active design variant
        designCount: 5,    // Total number of designs

        // Design-specific state (reset on switch)
        cardFlipped: false,
        translationVisible: false,
        revealStep: 0,
        // ...
    }
}

// Reset design-specific state when switching
watch: {
    currentDesign() {
        this.resetDesignState();
    }
}

methods: {
    resetDesignState() {
        // Reset all design-specific UI state
        this.cardFlipped = false;
        this.translationVisible = false;
        this.revealStep = 0;
        // Important: DON'T reset user progress or feedback!
    }
}
```

**HTML Pattern:**

```html
<!-- Each design as conditional block -->
<div v-if="currentDesign === 1">
    <!-- Design 1: Modal View -->
    <!-- Full implementation here -->
</div>

<div v-if="currentDesign === 2">
    <!-- Design 2: Inline Expansion -->
    <!-- Full implementation here -->
</div>

<!-- etc... -->
```

**Why not v-show?** Using `v-if` ensures each design is fully re-rendered when activated, preventing state leakage between designs.

### 2. Switcher UI (Toolbar)

**Essential Features:**
- Clear indication of current design
- Easy switching mechanism (dropdown, buttons, keyboard shortcuts)
- Per-design feedback collection
- Progress/usage stats
- Collapsible to reduce clutter
- Draggable for flexible positioning

**Good UX Principles:**

```html
<!-- Floating toolbar pattern -->
<div class="fixed z-50"
     :style="toolbarPosition">

    <!-- Draggable header (visual affordance) -->
    <div @mousedown="startDrag"
         class="cursor-move select-none bg-blue-600 text-white">
        <span>Design Prototype</span>
        <button @click="collapsed = !collapsed">☰</button>
    </div>

    <!-- Collapsible content -->
    <div v-if="!collapsed">
        <!-- Design selector -->
        <select v-model.number="currentDesign">
            <option value="1">1. Modal View</option>
            <option value="2">2. Inline Expansion</option>
            <!-- Clear, descriptive names! -->
        </select>

        <!-- Feedback for CURRENT design -->
        <textarea v-model="designFeedback[currentDesign]"
                  placeholder="Your thoughts on this design...">
        </textarea>

        <!-- Stats -->
        <div class="text-xs">
            Progress: {{ currentIndex + 1 }} / {{ totalItems }}
        </div>

        <!-- Export functionality -->
        <button @click="exportFeedback">
            Export All Feedback
        </button>
    </div>
</div>
```

**Visual Design:**
- Use distinct color (e.g., blue) separate from app design
- Always visible but non-intrusive
- Clear visual hierarchy (selector > feedback > actions)
- Mobile-responsive (stack vertically, larger touch targets)

### 3. Feedback Collection System

**Per-Design Feedback Structure:**

```javascript
designFeedback: {
    "1": "",  // Feedback for Design 1
    "2": "",  // Feedback for Design 2
    "3": "",  // etc...
    "4": "",
    "5": ""
}

// Always show feedback for CURRENT design
<textarea v-model="designFeedback[currentDesign]">
```

**Why per-design?**
- User can provide specific feedback as they test each design
- Feedback tied to context (they remember what they just tried)
- Prevents mixing feedback for different approaches
- Easier to analyze later ("Design 2 got positive feedback")

**Auto-save Pattern:**

```javascript
// Save to localStorage on every input
<textarea v-model="designFeedback[currentDesign]"
          @input="saveToLocalStorage">

// Method
saveToLocalStorage() {
    const data = {
        currentDesign: this.currentDesign,
        designFeedback: this.designFeedback,
        // ... other state
    };
    localStorage.setItem('appPrototype', JSON.stringify(data));
}
```

**Benefits:**
- Never lose feedback (even if browser crashes)
- Seamless experience across sessions
- User can close/reopen without losing work

### 4. Data Export

**Export Format (JSON):**

```json
{
  "timestamp": "2025-10-31T12:00:00.000Z",
  "currentDesign": 2,
  "sessionData": {
    "itemsReviewed": 15,
    "lastVisit": 1730376000000
  },
  "allFeedback": [
    {
      "design": 1,
      "designName": "Modal View",
      "feedback": "Easy to use but takes up too much screen space..."
    },
    {
      "design": 2,
      "designName": "Inline Expansion",
      "feedback": "Love the click-to-reveal interaction!"
    }
  ],
  "designFeedback": {
    "1": "...",
    "2": "..."
  },
  "progress": {
    "currentWordIndex": 14,
    "totalWords": 15
  }
}
```

**Export Implementation:**

```javascript
exportFeedback() {
    const exportData = {
        timestamp: new Date().toISOString(),
        currentDesign: this.currentDesign,

        // Formatted feedback
        allFeedback: Object.entries(this.designFeedback)
            .filter(([_, feedback]) => feedback.trim())
            .map(([design, feedback]) => ({
                design: parseInt(design),
                designName: this.getDesignName(parseInt(design)),
                feedback: feedback
            })),

        // Raw data
        designFeedback: this.designFeedback,

        // Context
        sessionData: this.sessionData,
        progress: {
            currentItemIndex: this.currentItemIndex,
            totalItems: this.items.length
        }
    };

    // Download as JSON
    const blob = new Blob(
        [JSON.stringify(exportData, null, 2)],
        { type: 'application/json' }
    );
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `feedback-${Date.now()}.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
}
```

### 5. State Persistence

**What to Persist:**

```javascript
localStorage.setItem('appPrototype', JSON.stringify({
    // Design state
    currentDesign: 1,

    // Feedback (most important!)
    designFeedback: {...},

    // UI preferences
    toolbarCollapsed: false,
    toolbarPosition: {x: 100, y: 50},

    // User progress (optional)
    currentItemIndex: 5,
    sessionData: {...}
}));
```

**What NOT to Persist:**
- Design-specific UI state (cardFlipped, revealStep, etc.)
  - These should reset when switching designs
- Temporary/transient state
- Computed values (can be recalculated)

**Load Pattern:**

```javascript
mounted() {
    this.loadFromLocalStorage();
}

loadFromLocalStorage() {
    const saved = localStorage.getItem('appPrototype');
    if (saved) {
        try {
            const data = JSON.parse(saved);
            this.currentDesign = data.currentDesign || 1;
            this.designFeedback = data.designFeedback || {};
            this.toolbarPosition = data.toolbarPosition || {x: null, y: null};
            // ...
        } catch (e) {
            console.error('Error loading state:', e);
            // Graceful fallback to defaults
        }
    }
}
```

## Advanced Features

### Draggable Toolbar

**Why:**
- Different screen sizes need different positions
- User preference (some prefer top, some bottom)
- Get toolbar out of the way of content being tested

**Implementation:**

```javascript
data() {
    return {
        isDragging: false,
        dragOffset: {x: 0, y: 0},
        toolbarPosition: {x: null, y: null}  // null = use CSS default
    }
}

methods: {
    startDrag(event) {
        this.isDragging = true;
        const rect = event.currentTarget.parentElement.getBoundingClientRect();
        this.dragOffset.x = event.clientX - rect.left;
        this.dragOffset.y = event.clientY - rect.top;

        document.addEventListener('mousemove', this.drag);
        document.addEventListener('mouseup', this.stopDrag);
    },

    drag(event) {
        if (!this.isDragging) return;
        this.toolbarPosition.x = event.clientX - this.dragOffset.x;
        this.toolbarPosition.y = event.clientY - this.dragOffset.y;
    },

    stopDrag() {
        this.isDragging = false;
        document.removeEventListener('mousemove', this.drag);
        document.removeEventListener('mouseup', this.stopDrag);
        this.saveToLocalStorage();  // Persist position
    }
}
```

```html
<!-- Conditional positioning -->
<div class="z-50"
     :class="toolbarPosition.x === null ? 'fixed top-4 right-4' : 'fixed'"
     :style="toolbarPosition.x !== null ?
             {left: toolbarPosition.x + 'px', top: toolbarPosition.y + 'px'} :
             {}">

    <div @mousedown="startDrag"
         class="cursor-move select-none">
        Drag me!
    </div>
</div>
```

**UX Tips:**
- Use `cursor: move` to indicate draggability
- Use `user-select: none` to prevent text selection while dragging
- Stop event propagation on buttons within draggable area
- Persist position so it stays where user placed it

### Keyboard Shortcuts

**Power User Feature:**

```javascript
mounted() {
    window.addEventListener('keydown', this.handleKeyboard);
}

beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyboard);
}

methods: {
    handleKeyboard(event) {
        // Alt + number to switch designs
        if (event.altKey && event.key >= '1' && event.key <= '5') {
            event.preventDefault();
            this.currentDesign = parseInt(event.key);
        }

        // Alt + E to export
        if (event.altKey && event.key === 'e') {
            event.preventDefault();
            this.exportFeedback();
        }
    }
}
```

### Usage Analytics

Track which designs users spend time on:

```javascript
data() {
    return {
        designUsage: {
            "1": 0,  // seconds spent on Design 1
            "2": 0,  // etc...
        },
        currentDesignStartTime: Date.now()
    }
}

watch: {
    currentDesign(newDesign, oldDesign) {
        // Record time spent on previous design
        const timeSpent = Date.now() - this.currentDesignStartTime;
        this.designUsage[oldDesign] += timeSpent / 1000;  // convert to seconds

        // Reset timer for new design
        this.currentDesignStartTime = Date.now();

        this.resetDesignState();
    }
}

// Include in export
exportFeedback() {
    const exportData = {
        // ... other data ...

        analytics: {
            totalTime: Object.values(this.designUsage).reduce((a,b) => a+b, 0),
            timePerDesign: this.designUsage,
            mostUsedDesign: this.getMostUsedDesign()
        }
    };
}
```

## Design Best Practices

### 1. Keep Designs Truly Independent

❌ **Bad:**
```javascript
// Shared state that might not work for all designs
data() {
    return {
        showTranslation: true,  // Design 3 doesn't have translation!
        animationStyle: 'fade'  // Design 1 uses flip!
    }
}
```

✅ **Good:**
```javascript
data() {
    return {
        // Design-specific state
        design1State: { cardFlipped: false },
        design2State: { translationVisible: false },
        design3State: { revealStep: 0 },

        // Or simpler: individual flags
        cardFlipped: false,       // For Design 1
        translationVisible: false, // For Design 2
        revealStep: 0,            // For Design 3
    }
}
```

### 2. Use Descriptive Design Names

❌ **Bad:**
```html
<option value="1">Design A</option>
<option value="2">Design B</option>
```

✅ **Good:**
```html
<option value="1">1. Modal View</option>
<option value="2">2. Inline Expansion</option>
<option value="3">3. Sidebar Navigation</option>
```

**Benefits:**
- User remembers which design is which
- Feedback mentions specific design features
- Easier to discuss ("I like the Modal View better...")

### 3. Provide Visual Feedback

**Show active design:**

```html
<!-- Title shows current design -->
<div class="text-center mb-4">
    <h2 class="text-2xl font-bold">{{ designNames[currentDesign] }}</h2>
    <p class="text-sm text-gray-600">{{ designDescriptions[currentDesign] }}</p>
</div>

<!-- Toolbar shows active design -->
<div class="bg-blue-600 text-white">
    Design {{ currentDesign }}/{{ designCount }}
</div>
```

### 4. Make Switching Easy

**Multiple methods:**
- Dropdown (precise selection)
- Next/Previous buttons (linear exploration)
- Keyboard shortcuts (power users)
- URL parameters (sharing specific design: `?design=2`)

```javascript
// URL parameter support
mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    const designParam = urlParams.get('design');
    if (designParam && designParam >= 1 && designParam <= this.designCount) {
        this.currentDesign = parseInt(designParam);
    }
    this.loadFromLocalStorage();
}

// Update URL when design changes
watch: {
    currentDesign(newDesign) {
        const url = new URL(window.location);
        url.searchParams.set('design', newDesign);
        window.history.pushState({}, '', url);
    }
}
```

## Common Pitfalls

### 1. State Leakage Between Designs

**Problem:** Design 1 state affects Design 2

**Solution:** Reset all design-specific state when switching

```javascript
resetDesignState() {
    // Reset EVERYTHING that's design-specific
    this.cardFlipped = false;
    this.translationVisible = false;
    this.revealStep = 0;
    this.minimalistPhase = 1;
    // Don't reset: currentWordIndex, feedback, etc.
}
```

### 2. Losing User Feedback

**Problem:** User writes feedback, switches designs, comes back - feedback gone!

**Solution:** Auto-save on input + per-design storage

```html
<textarea v-model="designFeedback[currentDesign]"
          @input="saveToLocalStorage">
```

### 3. Inconsistent Data Across Designs

**Problem:** Design 1 shows different data than Design 2

**Solution:** Use same data source for all designs

```javascript
// Good: shared data
computed: {
    currentItem() {
        return this.items[this.currentIndex];
    }
}

// All designs use the same data: {{ currentItem.title }}, {{ currentItem.description }}, etc.
```

### 4. Poor Mobile Experience

**Problem:** Toolbar overlaps content, dropdown too small

**Solution:** Responsive design

```html
<div class="fixed top-4 right-4 z-50
            md:w-80 w-full max-w-sm">
    <select class="w-full px-3 py-3 text-base"> <!-- Larger touch target -->
    </select>
</div>
```

## Metrics & Analysis

### What to Track

**Quantitative:**
- Time spent on each design
- Number of switches between designs
- Completion rate per design (did they finish testing it?)
- Click/interaction patterns

**Qualitative:**
- Written feedback (most valuable!)
- Which designs got the most feedback (engagement)
- Sentiment analysis of feedback text
- Feature mentions ("I like the highlighted word")

### Sample Analysis

```javascript
// Process exported feedback
function analyzeFeedback(exportData) {
    const feedback = exportData.allFeedback;

    // Designs with feedback
    const providedFeedback = feedback.filter(f => f.feedback.trim());
    console.log(`${providedFeedback.length}/${feedback.length} designs got feedback`);

    // Sentiment analysis (simple)
    const positive = feedback.filter(f =>
        f.feedback.toLowerCase().includes('good') ||
        f.feedback.toLowerCase().includes('like') ||
        f.feedback.toLowerCase().includes('love')
    );

    const negative = feedback.filter(f =>
        f.feedback.toLowerCase().includes('bad') ||
        f.feedback.toLowerCase().includes('don\'t like') ||
        f.feedback.toLowerCase().includes('confusing')
    );

    // Most engaged design
    const mostFeedback = feedback.reduce((max, f) =>
        f.feedback.length > max.feedback.length ? f : max
    );

    return {
        engagementRate: providedFeedback.length / feedback.length,
        positiveCount: positive.length,
        negativeCount: negative.length,
        mostEngagingDesign: mostFeedback.designName
    };
}
```

## Checklist

Before deploying a design switcher:

- [ ] All designs use the same data/content
- [ ] Design-specific state resets when switching
- [ ] Per-design feedback collection implemented
- [ ] Auto-save to localStorage on every input
- [ ] Export functionality with timestamp
- [ ] Clear visual indication of current design
- [ ] Toolbar is draggable (or fixed in good position)
- [ ] Mobile-responsive design
- [ ] Keyboard shortcuts (optional but nice)
- [ ] URL parameter support for sharing
- [ ] Reset/clear data functionality
- [ ] Tested switching between all design combinations
- [ ] Verified localStorage persistence works
- [ ] Export includes all relevant data
- [ ] Design names are descriptive, not generic

## Conclusion

A well-implemented design switcher enables rapid iteration and data-driven decision making during the prototyping phase. The key is making it easy to:
1. Switch between designs
2. Provide feedback
3. Preserve state
4. Export data

Keep the switcher UI simple, non-intrusive, and focused on collecting quality feedback from users.
