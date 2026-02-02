# Phase 05: Mobile Optimization

**Priority:** P1
**Status:** Pending
**Effort:** 2h
**Dependencies:** Phase 03, 04 complete

---

## Context Links

- [Research: Mobile UX](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-mobile-3d-viewer-ux.md)
- [Phase 04 Controls](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/phase-04-enhanced-controls.md)

---

## Overview

Add mobile-first tutorial overlay showing gestures on first visit, optimize touch responsiveness, prevent scroll interference, ensure 30+ FPS performance.

---

## Key Insights

- **85% of users** try dragging within 3 seconds
- **Only 40%** discover pinch-to-zoom without tutorial
- Tutorial overlay reduces confusion by 60%+
- First-time hints stored in localStorage
- Touch action `pan-y` prevents scroll blocking

---

## Requirements

### Functional
- [ ] Tutorial overlay on first visit (mobile only)
- [ ] Animated gesture hints (drag, pinch)
- [ ] Auto-dismiss after 5s or first interaction
- [ ] localStorage tracking (show once per browser)
- [ ] Touch optimization (no lag, 30+ FPS)

### Non-Functional
- [ ] Tutorial visible but non-intrusive
- [ ] Animation smooth (CSS transitions)
- [ ] No performance degradation
- [ ] Accessible (can be dismissed with keyboard)

---

## Architecture

### Tutorial Overlay Structure
```html
<div class="model-viewer-tutorial" data-tutorial-overlay>
  <div class="tutorial-backdrop"></div>
  <div class="tutorial-content">
    <div class="tutorial-gesture drag-gesture">
      <svg class="hand-icon"><!-- Animated hand --></svg>
      <p>Drag to rotate</p>
    </div>
    <div class="tutorial-gesture pinch-gesture">
      <svg class="hand-icon"><!-- Animated pinch --></svg>
      <p>Pinch to zoom</p>
    </div>
    <button class="tutorial-dismiss" aria-label="Dismiss tutorial">
      Got it
    </button>
  </div>
</div>
```

### Tutorial Flow
```
Page Load (Mobile)
  ↓
Check localStorage['3d-viewer-tutorial-seen']
  ↓
Not seen? → Show tutorial overlay (5s auto-dismiss)
  ↓
User interacts OR clicks "Got it" OR 5s timeout
  ↓
Set localStorage, hide tutorial
```

---

## Related Code Files

### To Create
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-mobile-tutorial.js` (tutorial logic)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-3d-viewer-tutorial-overlay.liquid` (HTML template)

### To Modify
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid` (include tutorial)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhancements.css` (tutorial styles)

---

## Implementation Steps

### 1. Create Tutorial Liquid Snippet
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-3d-viewer-tutorial-overlay.liquid`

```liquid
{%- comment -%}
  Tutorial overlay for mobile 3D viewer
  Shows gesture hints on first visit
{%- endcomment -%}

<div
  class="model-viewer-tutorial"
  data-tutorial-overlay
  role="dialog"
  aria-labelledby="tutorial-title"
  aria-modal="true"
  hidden>

  <div class="tutorial-backdrop"></div>

  <div class="tutorial-content">
    <h2 id="tutorial-title" class="sr-only">3D Viewer Controls</h2>

    <div class="tutorial-gestures">
      {%- comment -%} Drag Gesture {%- endcomment -%}
      <div class="tutorial-gesture drag-gesture">
        <svg width="80" height="80" viewBox="0 0 80 80" class="gesture-icon">
          <circle cx="40" cy="40" r="12" fill="white" opacity="0.9">
            <animateTransform
              attributeName="transform"
              attributeType="XML"
              type="translate"
              values="0,0; 20,0; 0,0"
              dur="2s"
              repeatCount="indefinite"/>
          </circle>
          <path d="M40 40 L60 40" stroke="white" stroke-width="2" opacity="0.6">
            <animate
              attributeName="opacity"
              values="0.6;1;0.6"
              dur="2s"
              repeatCount="indefinite"/>
          </path>
        </svg>
        <p class="gesture-label">Drag to rotate</p>
      </div>

      {%- comment -%} Pinch Gesture {%- endcomment -%}
      <div class="tutorial-gesture pinch-gesture">
        <svg width="80" height="80" viewBox="0 0 80 80" class="gesture-icon">
          <circle cx="30" cy="40" r="10" fill="white" opacity="0.9">
            <animate
              attributeName="cx"
              values="30;20;30"
              dur="2s"
              repeatCount="indefinite"/>
          </circle>
          <circle cx="50" cy="40" r="10" fill="white" opacity="0.9">
            <animate
              attributeName="cx"
              values="50;60;50"
              dur="2s"
              repeatCount="indefinite"/>
          </circle>
        </svg>
        <p class="gesture-label">Pinch to zoom</p>
      </div>
    </div>

    <button
      type="button"
      class="tutorial-dismiss"
      data-tutorial-dismiss
      aria-label="Dismiss tutorial and start exploring">
      Got it
    </button>
  </div>
</div>
```

### 2. Create Tutorial JavaScript
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-mobile-tutorial.js`

```javascript
/**
 * Mobile Tutorial Overlay for 3D Viewer
 * Shows gesture hints on first visit (mobile only)
 */

class Product3DViewerTutorial {
  constructor() {
    this.tutorialKey = '3d-viewer-tutorial-seen';
    this.tutorialOverlay = document.querySelector('[data-tutorial-overlay]');
    this.dismissButton = document.querySelector('[data-tutorial-dismiss]');
    this.modelViewer = document.querySelector('model-viewer');
    this.autoHideTimeout = null;

    if (this.shouldShowTutorial()) {
      this.init();
    }
  }

  shouldShowTutorial() {
    // Only show on mobile devices
    const isMobile = window.innerWidth <= 768;
    if (!isMobile) return false;

    // Check if tutorial already seen
    const hasSeenTutorial = localStorage.getItem(this.tutorialKey);
    if (hasSeenTutorial) return false;

    // Check if elements exist
    return this.tutorialOverlay && this.modelViewer;
  }

  init() {
    this.showTutorial();
    this.setupEventListeners();
    this.setupAutoHide();
  }

  showTutorial() {
    this.tutorialOverlay.removeAttribute('hidden');
    this.tutorialOverlay.classList.add('tutorial-visible');

    // Prevent body scroll while tutorial visible
    document.body.style.overflow = 'hidden';

    // Focus dismiss button for accessibility
    setTimeout(() => {
      this.dismissButton?.focus();
    }, 100);
  }

  hideTutorial() {
    this.tutorialOverlay.classList.remove('tutorial-visible');
    this.tutorialOverlay.classList.add('tutorial-hidden');

    // Restore body scroll
    document.body.style.overflow = '';

    // Remove from DOM after animation
    setTimeout(() => {
      this.tutorialOverlay.setAttribute('hidden', '');
    }, 300);

    // Mark as seen
    this.markAsSeen();

    // Clear auto-hide timeout
    if (this.autoHideTimeout) {
      clearTimeout(this.autoHideTimeout);
    }
  }

  markAsSeen() {
    try {
      localStorage.setItem(this.tutorialKey, 'true');
    } catch (e) {
      console.warn('Could not save tutorial state to localStorage');
    }
  }

  setupEventListeners() {
    // Dismiss button click
    this.dismissButton?.addEventListener('click', () => {
      this.hideTutorial();
    });

    // Hide on first model interaction
    const hideOnInteraction = () => {
      this.hideTutorial();
      this.modelViewer.removeEventListener('mousedown', hideOnInteraction);
      this.modelViewer.removeEventListener('touchstart', hideOnInteraction);
    };

    this.modelViewer.addEventListener('mousedown', hideOnInteraction);
    this.modelViewer.addEventListener('touchstart', hideOnInteraction);

    // Keyboard: Escape to dismiss
    const handleKeydown = (e) => {
      if (e.key === 'Escape') {
        this.hideTutorial();
        document.removeEventListener('keydown', handleKeydown);
      }
    };
    document.addEventListener('keydown', handleKeydown);
  }

  setupAutoHide() {
    // Auto-hide after 5 seconds
    this.autoHideTimeout = setTimeout(() => {
      this.hideTutorial();
    }, 5000);
  }
}

// Initialize when DOM ready
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', () => {
    new Product3DViewerTutorial();
  });
} else {
  new Product3DViewerTutorial();
}
```

### 3. Add Tutorial Styles
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhancements.css`

```css
/* Tutorial Overlay */
.model-viewer-tutorial {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.model-viewer-tutorial[hidden] {
  display: none;
}

.model-viewer-tutorial.tutorial-visible {
  opacity: 1;
}

.model-viewer-tutorial.tutorial-hidden {
  opacity: 0;
}

/* Tutorial Backdrop */
.tutorial-backdrop {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(4px);
}

/* Tutorial Content */
.tutorial-content {
  position: relative;
  z-index: 1;
  background: white;
  border-radius: 16px;
  padding: 32px 24px;
  max-width: 320px;
  text-align: center;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

/* Tutorial Gestures */
.tutorial-gestures {
  display: flex;
  justify-content: space-around;
  margin-bottom: 24px;
}

.tutorial-gesture {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.gesture-icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 16px;
  padding: 12px;
}

.gesture-label {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin: 0;
}

/* Dismiss Button */
.tutorial-dismiss {
  width: 100%;
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.tutorial-dismiss:hover {
  transform: scale(1.02);
}

.tutorial-dismiss:active {
  transform: scale(0.98);
}

/* Desktop: Hide tutorial (mobile only) */
@media (min-width: 769px) {
  .model-viewer-tutorial {
    display: none !important;
  }
}

/* Small mobile optimization */
@media (max-width: 380px) {
  .tutorial-content {
    max-width: 280px;
    padding: 24px 16px;
  }

  .gesture-icon {
    width: 60px;
    height: 60px;
  }

  .gesture-label {
    font-size: 12px;
  }
}
```

### 4. Include Tutorial in product-media.liquid
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid`

**Add after model-viewer closing tag:**
```liquid
{%- when 'model' -%}
  <product-3d-viewer-controls class="product-3d-viewer-wrapper">
    <model-viewer ...>
      {%- comment -%} ...existing content... {%- endcomment -%}
    </model-viewer>

    {%- comment -%} ...existing buttons... {%- endcomment -%}

    {%- comment -%} Tutorial Overlay (Mobile Only) {%- endcomment -%}
    {% render 'product-3d-viewer-tutorial-overlay' %}
  </product-3d-viewer-controls>
```

### 5. Include JavaScript in Theme
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/layout/theme.liquid`

**Add before `</body>`:**
```liquid
<script src="{{ 'product-3d-viewer-mobile-tutorial.js' | asset_url }}" defer></script>
```

### 6. Optimize Touch Performance
**Verify in product-media.liquid:**
```liquid
<model-viewer
  touch-action="pan-y"  <!-- Allows vertical scroll -->
  interaction-prompt="none"  <!-- Disable built-in prompt -->
  ...
>
```

### 7. Test Mobile Performance
```bash
# Test on real devices:
# iPhone 12 (iOS Safari)
# Galaxy A52 (Android Chrome)
# Budget Android (2-3 years old)

# Use Chrome DevTools:
# 1. Enable CPU throttling (4x slowdown)
# 2. Network throttling (Fast 3G)
# 3. Check FPS in Performance panel (target: 30+)
```

---

## Todo List

- [ ] Create product-3d-viewer-tutorial-overlay.liquid snippet
- [ ] Design animated gesture SVGs (drag, pinch)
- [ ] Create product-3d-viewer-mobile-tutorial.js module
- [ ] Implement localStorage check logic
- [ ] Add tutorial show/hide animations
- [ ] Implement auto-hide after 5s
- [ ] Add dismiss button functionality
- [ ] Hide on first model interaction
- [ ] Add keyboard dismiss (Escape)
- [ ] Update CSS with tutorial styles
- [ ] Include tutorial in product-media.liquid
- [ ] Include JavaScript in theme.liquid
- [ ] Test tutorial shows on mobile (first visit)
- [ ] Test tutorial doesn't show on desktop
- [ ] Test localStorage persists across sessions
- [ ] Test auto-hide after 5s
- [ ] Test dismiss button
- [ ] Test Escape key dismissal
- [ ] Verify touch-action allows page scroll
- [ ] Test on real mobile devices (iOS, Android)
- [ ] Measure FPS during rotation (target: 30+)
- [ ] Test on slow devices (CPU throttling)

---

## Success Criteria

✅ Tutorial shows on mobile (first visit only)
✅ Tutorial doesn't show on desktop
✅ Animated gesture hints visible (drag, pinch)
✅ Auto-dismisses after 5s
✅ Dismisses on first interaction
✅ "Got it" button dismisses tutorial
✅ Escape key dismisses tutorial
✅ localStorage prevents re-showing
✅ 30+ FPS during rotation (mobile)
✅ No scroll blocking (touch-action: pan-y)
✅ Accessible (keyboard, screen reader)

---

## Risk Assessment

**Medium Risk** - Mobile performance critical

**Potential Issues:**
- Low FPS on budget devices → Test with CPU throttling
- Tutorial blocks critical content → Semi-transparent backdrop
- localStorage not supported → Graceful fallback
- Touch conflicts with page scroll → Proper touch-action setting

**Mitigation:**
- Test on 2-3 year old mid-range phones (real devices)
- Progressive loading (low-poly first, full quality after)
- Feature detection for localStorage
- Verify touch-action="pan-y" allows scroll
- Monitor Lighthouse performance scores

---

## Security Considerations

**localStorage:**
- Only stores boolean flag (no sensitive data)
- Try/catch for privacy mode browsers
- No cross-site data leakage

**Performance:**
- Lazy load tutorial JavaScript (defer attribute)
- CSS animations GPU-accelerated
- No memory leaks (event listeners cleaned up)

---

## Next Steps

After mobile optimization:
→ **Phase 06:** Cross-device testing
→ Performance profiling (Lighthouse, WebPageTest)
→ User acceptance testing on real devices
