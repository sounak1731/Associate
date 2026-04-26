# Phase 2: Major UI Enhancements Plan

## Overview
Expanding sidebar, restructuring matter view, transforming document editor, and establishing consistent priority color scheme.

---

## Change 1: Expandable Sidebar (ChatGPT-Style)

### Current Behavior
- Narrow 48px rail with single-letter dots
- Tooltips on hover

### New Behavior
- Default: Expanded 220px sidebar showing full matter names
- Collapse toggle: Shrinks to current 48px compact mode
- Smooth CSS transition between states
- Persistent state in localStorage

### CSS Additions
```css
/* Expanded Sidebar */
.matters-rail.expanded {
  width: 220px;
  align-items: stretch;
  padding: 14px 12px;
}
.matters-rail.expanded .brand-mark {
  width: auto;
  height: auto;
  padding: 8px 12px;
  justify-content: flex-start;
  gap: 10px;
}
.matters-rail.expanded .brand-mark::after {
  content: 'ssociate';
  font-size: 15px;
}
.matters-rail.expanded .matter-dot {
  width: auto;
  height: auto;
  padding: 8px 12px;
  justify-content: flex-start;
  gap: 10px;
  font-size: 11px;
}
.matters-rail.expanded .matter-dot .dot-label {
  font-weight: 600;
  min-width: 24px;
}
.matters-rail.expanded .matter-dot .matter-name {
  font-weight: 400;
  opacity: 0.9;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.matters-rail.expanded .collapse-btn {
  display: flex;
}

/* Collapse Button */
.collapse-btn {
  display: none;
  width: 24px;
  height: 24px;
  border-radius: 6px;
  background: var(--paper-deep);
  border: 1px solid var(--rule);
  cursor: pointer;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
  transition: all 0.15s;
}
.collapse-btn:hover {
  background: var(--ink);
  color: var(--paper);
  border-color: var(--ink);
}

/* Compact Mode (current) */
.matters-rail.compact {
  width: 48px;
  min-width: 48px;
}
.matters-rail.compact .matter-dot .matter-name {
  display: none;
}
```

### HTML Structure Changes
```html
<aside class="matters-rail expanded" id="mattersRail">
  <button class="collapse-btn" onclick="toggleSidebar()" title="Collapse sidebar">
    <svg><!-- chevron-left icon --></svg>
  </button>
  
  <div class="brand-mark active" onclick="goHome()" title="Home">
    <span class="dot-label">A</span>
  </div>
  <div class="rail-separator"></div>

  <div class="matter-dot type-litigation urgent" onclick="openMatter('krishnamurthy')">
    <span class="dot-label">KV</span>
    <span class="matter-name">Krishnamurthy v. Karnataka</span>
  </div>
  <!-- etc -->
</aside>
```

### JavaScript
```javascript
let sidebarExpanded = localStorage.getItem('sidebarExpanded') !== 'false';

function initSidebar() {
  const rail = document.getElementById('mattersRail');
  if (sidebarExpanded) {
    rail.classList.add('expanded');
    rail.classList.remove('compact');
  } else {
    rail.classList.add('compact');
    rail.classList.remove('expanded');
  }
}

function toggleSidebar() {
  sidebarExpanded = !sidebarExpanded;
  localStorage.setItem('sidebarExpanded', sidebarExpanded);
  initSidebar();
}
```

---

## Change 2: Next Steps First in Matter View

### Current Order
1. Agent Brief (intro)
2. AI Warning Banner
3. What's Happened (chronology)
4. Agent Aside
5. Open Issues
6. **Next Steps** ← currently last

### New Order
1. Agent Brief (intro)
2. AI Warning Banner
3. **Next Steps** ← moved to #3
4. What's Happened (chronology)
5. Agent Aside
6. Open Issues

### Implementation
In `renderMatterView()` function, reorder the HTML template:
```javascript
brief.innerHTML = `
  <div class="matter-view-header">...</div>
  
  <section class="intro">
    <div class="intro-label">...</div>
    <p class="intro-body">${m.narrative}</p>
  </section>

  <!-- AI Warning Banner -->
  <div class="ai-warning-banner">...</div>

  <!-- NEXT STEPS - NOW FIRST -->
  <section class="section">
    <div class="section-head"><span>Next steps</span></div>
    <div class="sugg-grid" id="suggGrid">${suggCards}</div>
  </section>

  <!-- Then chronology, issues, etc -->
  <section class="section">
    <div class="section-head"><span>What's happened</span></div>
    ...
  </section>
  ...
`;
```

---

## Change 3: Document Editor Overhaul

### Remove Track Changes Button
Find and remove from doc-top-right:
```html
<!-- REMOVE THIS -->
<button class="doc-tool" id="trackChangesBtn" onclick="toggleTrackChanges()">Track changes</button>
```

### New Document Mode Dropdown
Add to top-right toolbar:
```html
<div class="doc-mode-dropdown" id="docModeDropdown">
  <button class="doc-mode-trigger" onclick="toggleDocModeMenu()">
    <span id="currentDocMode">Viewing</span>
    <svg><!-- chevron-down --></svg>
  </button>
  <div class="doc-mode-menu" id="docModeMenu">
    <div class="doc-mode-option" data-mode="viewing" onclick="setDocMode('viewing')">
      <svg><!-- eye icon --></svg>
      <span>Viewing</span>
      <small>Read only</small>
    </div>
    <div class="doc-mode-option" data-mode="editing" onclick="setDocMode('editing')">
      <svg><!-- edit icon --></svg>
      <span>Editing</span>
      <small>Make changes directly</small>
    </div>
    <div class="doc-mode-option" data-mode="commenting" onclick="setDocMode('commenting')">
      <svg><!-- comment icon --></svg>
      <span>Commenting</span>
      <small>Suggest changes as comments</small>
    </div>
  </div>
</div>
```

### CSS for Dropdown
```css
.doc-mode-dropdown {
  position: relative;
}
.doc-mode-trigger {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border: 1px solid var(--rule);
  border-radius: 6px;
  background: var(--paper);
  font-family: 'Inter Tight', sans-serif;
  font-size: 12px;
  font-weight: 500;
  color: var(--ink-2);
  cursor: pointer;
  transition: all 0.12s;
}
.doc-mode-trigger:hover {
  background: var(--paper-deep);
  color: var(--ink);
}
.doc-mode-menu {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 6px;
  background: var(--paper);
  border: 1px solid var(--rule);
  border-radius: 8px;
  box-shadow: 0 10px 40px rgba(0,0,0,0.15);
  min-width: 180px;
  display: none;
  z-index: 100;
}
.doc-mode-menu.open {
  display: block;
  animation: menuIn 0.2s ease-out;
}
@keyframes menuIn {
  from { opacity: 0; transform: translateY(-8px); }
  to { opacity: 1; transform: translateY(0); }
}
.doc-mode-option {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  cursor: pointer;
  transition: background 0.12s;
  border-bottom: 1px solid var(--rule-soft);
}
.doc-mode-option:last-child {
  border-bottom: none;
}
.doc-mode-option:hover {
  background: var(--paper-deep);
}
.doc-mode-option.active {
  background: var(--sage-soft);
}
.doc-mode-option svg {
  width: 16px;
  height: 16px;
  color: var(--ink-3);
}
.doc-mode-option span {
  font-family: 'Inter Tight', sans-serif;
  font-size: 12px;
  font-weight: 500;
  color: var(--ink);
}
.doc-mode-option small {
  margin-left: auto;
  font-size: 10px;
  color: var(--ink-3);
}
```

### Commenting Mode Behavior
When `docMode === 'commenting'`:
- Selecting text shows "Add comment" popup
- Typing creates comment box in right margin
- Comments are saved separately from document

```javascript
let currentDocMode = 'viewing';

function setDocMode(mode) {
  currentDocMode = mode;
  document.getElementById('currentDocMode').textContent = 
    mode.charAt(0).toUpperCase() + mode.slice(1);
  
  // Update active state in menu
  document.querySelectorAll('.doc-mode-option').forEach(opt => {
    opt.classList.toggle('active', opt.dataset.mode === mode);
  });
  
  // Apply mode-specific behaviors
  const docPage = document.getElementById('docPageContent');
  docPage.contentEditable = (mode === 'editing');
  
  if (mode === 'commenting') {
    enableCommentingMode();
  } else {
    disableCommentingMode();
  }
  
  closeDocModeMenu();
}

function enableCommentingMode() {
  const docPage = document.getElementById('docPageContent');
  docPage.addEventListener('mouseup', handleTextSelection);
}

function handleTextSelection() {
  if (currentDocMode !== 'commenting') return;
  
  const selection = window.getSelection();
  if (selection.toString().trim().length > 0) {
    showCommentPopup(selection);
  }
}
```

---

## Change 4: AI Comments Toggle

### Toggle Button in Doc Toolbar
```html
<button class="doc-tool ai-comments-toggle" id="aiCommentsToggle" onclick="toggleAIComments()">
  <svg><!-- robot/AI icon --></svg>
  <span id="aiCommentsLabel">AI Comments: ON</span>
</button>
```

### CSS
```css
.ai-comments-toggle {
  display: flex;
  align-items: center;
  gap: 6px;
}
.ai-comments-toggle.inactive {
  opacity: 0.5;
}
.ai-comments-toggle.inactive svg {
  color: var(--ink-4);
}

/* AI Comment styling */
.margin-note.ai-comment {
  border-left: 3px solid var(--sage);
}
.margin-note.ai-comment.hidden {
  display: none;
}
.margin-note.ai-comment::before {
  content: 'AI';
  position: absolute;
  top: -8px;
  right: 8px;
  background: var(--sage);
  color: var(--paper);
  font-size: 8px;
  font-weight: 600;
  padding: 2px 5px;
  border-radius: 3px;
}
```

### JavaScript
```javascript
let aiCommentsVisible = localStorage.getItem('aiCommentsVisible') !== 'false';

function toggleAIComments() {
  aiCommentsVisible = !aiCommentsVisible;
  localStorage.setItem('aiCommentsVisible', aiCommentsVisible);
  updateAICommentsVisibility();
}

function updateAICommentsVisibility() {
  const toggle = document.getElementById('aiCommentsToggle');
  const label = document.getElementById('aiCommentsLabel');
  const aiNotes = document.querySelectorAll('.margin-note.ai-comment');
  
  if (aiCommentsVisible) {
    toggle?.classList.remove('inactive');
    if (label) label.textContent = 'AI Comments: ON';
    aiNotes.forEach(note => note.classList.remove('hidden'));
  } else {
    toggle?.classList.add('inactive');
    if (label) label.textContent = 'AI Comments: OFF';
    aiNotes.forEach(note => note.classList.add('hidden'));
  }
}

// Mark existing AI comments
function markAIComments() {
  document.querySelectorAll('.margin-note').forEach(note => {
    // Check if note contains AI-related text
    if (note.textContent.includes('Agent') || 
        note.textContent.includes('agent') ||
        note.classList.contains('ai-comment')) {
      note.classList.add('ai-comment');
    }
  });
}
```

---

## Change 5 & 6: Priority Color Scheme

### Color Definitions (Add to CSS variables)
```css
:root {
  /* Existing colors... */
  
  /* Priority Colors - Consistent Across App */
  --priority-high: #dc2626;      /* Red - Critical */
  --priority-high-soft: #fee2e2; /* Light red background */
  --priority-medium: #f97316;    /* Orange - Medium */
  --priority-medium-soft: #ffedd5; /* Light orange background */
  --priority-low: #eab308;       /* Yellow - Low */
  --priority-low-soft: #fef9c3;  /* Light yellow background */
}
```

### Priority Badge Components
```css
/* Priority Badge Base */
.priority-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-family: 'Inter Tight', sans-serif;
  font-size: 10px;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 4px;
  text-transform: uppercase;
  letter-spacing: 0.3px;
}

.priority-high {
  background: var(--priority-high-soft);
  color: var(--priority-high);
  border: 1px solid rgba(220, 38, 38, 0.2);
}

.priority-medium {
  background: var(--priority-medium-soft);
  color: #c2410c;
  border: 1px solid rgba(249, 115, 22, 0.2);
}

.priority-low {
  background: var(--priority-low-soft);
  color: #a16207;
  border: 1px solid rgba(234, 179, 8, 0.2);
}

/* Priority Indicators (dots, borders) */
.priority-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}
.priority-dot.high { background: var(--priority-high); }
.priority-dot.medium { background: var(--priority-medium); }
.priority-dot.low { background: var(--priority-low); }

/* Matter Row Priority Borders */
.matter-row.priority-high {
  border-left: 3px solid var(--priority-high);
}
.matter-row.priority-medium {
  border-left: 3px solid var(--priority-medium);
}
.matter-row.priority-low {
  border-left: 3px solid var(--priority-low);
}

/* Issue Tags using priority colors */
.issue-tag.critical {
  background: var(--priority-high-soft);
  color: var(--priority-high);
  border-color: rgba(220, 38, 38, 0.25);
}

.issue-tag.warning,
.issue-tag.medium {
  background: var(--priority-medium-soft);
  color: #c2410c;
  border-color: rgba(249, 115, 22, 0.25);
}

.issue-tag.low,
.issue-tag.monitor {
  background: var(--priority-low-soft);
  color: #a16207;
  border-color: rgba(234, 179, 8, 0.25);
}

/* Redlines Priority */
.rl-severity.critical {
  background: var(--priority-high-soft);
  color: var(--priority-high);
}
.rl-severity.amber {
  background: var(--priority-medium-soft);
  color: #c2410c;
}
.rl-severity.monitor {
  background: var(--priority-low-soft);
  color: #a16207;
}

/* Sidebar Matter Dot Urgency Indicators */
.matter-dot.urgent-high::after {
  background: var(--priority-high);
}
.matter-dot.urgent-medium::after {
  background: var(--priority-medium);
}
.matter-dot.urgent-low::after {
  background: var(--priority-low);
}
```

### Priority Legend Component
```html
<!-- Add to Home View or as floating legend -->
<div class="priority-legend">
  <div class="priority-legend-title">Priority Guide</div>
  <div class="priority-legend-item">
    <span class="priority-dot high"></span>
    <span>Critical / High</span>
  </div>
  <div class="priority-legend-item">
    <span class="priority-dot medium"></span>
    <span>Medium</span>
  </div>
  <div class="priority-legend-item">
    <span class="priority-dot low"></span>
    <span>Low / Monitor</span>
  </div>
</div>
```

```css
.priority-legend {
  position: fixed;
  bottom: 70px;
  right: 24px;
  background: var(--paper);
  border: 1px solid var(--rule);
  border-radius: 10px;
  padding: 14px 16px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
  z-index: 100;
}
.priority-legend-title {
  font-family: 'Inter Tight', sans-serif;
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: var(--ink-3);
  margin-bottom: 10px;
}
.priority-legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-family: 'Inter Tight', sans-serif;
  font-size: 11px;
  color: var(--ink-2);
  margin-bottom: 6px;
}
.priority-legend-item:last-child {
  margin-bottom: 0;
}
```

---

## Implementation Order

### Phase A: Priority Color Scheme (Foundation)
1. Add CSS variables for priority colors
2. Create priority badge classes
3. Update existing tags/labels to use priority classes
4. Add priority legend to home view

### Phase B: Sidebar Expansion
1. Restructure sidebar HTML with name spans
2. Add expanded/compact CSS classes
3. Add collapse button
4. Implement JavaScript toggle with localStorage
5. Update matter data to include priority levels

### Phase C: Matter View Reordering
1. Modify renderMatterView() to put Next Steps first
2. Test that all matter types render correctly

### Phase D: Document Editor Overhaul
1. Remove Track Changes button
2. Create document mode dropdown component
3. Implement mode switching logic
4. Add commenting mode behavior
5. Add AI comments toggle
6. Integrate priority colors into redlines

---

## Testing Checklist

- [ ] Sidebar expands/collapses smoothly
- [ ] Full matter names display correctly when expanded
- [ ] Next Steps appears first in matter view
- [ ] Document mode dropdown works (Viewing/Editing/Commenting)
- [ ] Commenting mode allows adding comments to text
- [ ] AI Comments toggle shows/hides AI margin notes
- [ ] Priority colors consistent across:
  - Matter list items
  - Issue tags
  - Redline severities
  - Sidebar urgency indicators
- [ ] Priority legend visible and helpful

---

*Plan created: 2026-04-26*
*Estimated effort: 3-4 hours*
