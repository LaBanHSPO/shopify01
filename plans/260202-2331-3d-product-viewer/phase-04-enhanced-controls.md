# Phase 04: Enhanced Controls

**Priority:** P2
**Status:** Pending
**Effort:** 2h
**Dependencies:** Phase 03 complete

---

## Context Links

- [Phase 03 Integration](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/phase-03-product-media-integration.md)
- [Research: Mobile UX](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-mobile-3d-viewer-ux.md)

---

## Overview

Add custom JavaScript controls for enhanced zoom/rotation: reset button, keyboard controls, zoom level indicator, smooth momentum damping.

---

## Key Insights

- Model-viewer has built-in JavaScript API for camera control
- Custom buttons can be added via slots or external HTML
- Keyboard accessibility required for WCAG 2.2 compliance
- Reset view button is essential (users get disoriented)
- Smooth damping creates physical feel

---

## Requirements

### Functional
- [ ] Reset button to return to default view
- [ ] Keyboard controls (arrow keys, +/- zoom)
- [ ] Zoom level indicator (optional)
- [ ] Smooth rotation momentum (damping)
- [ ] Prevent extreme camera angles

### Non-Functional
- [ ] Controls unobtrusive (minimal UI)
- [ ] Touch-friendly (44x44px minimum)
- [ ] Accessible (ARIA labels, focus indicators)
- [ ] Performance: <50ms response time

---

## Architecture

### JavaScript Module Structure
```javascript
// assets/product-3d-viewer-enhanced-controls.js

class Product3DViewerControls extends HTMLElement {
  constructor() {
    super();
    this.modelViewer = this.querySelector('model-viewer');
    this.resetButton = this.querySelector('[data-3d-reset]');
    this.setupEventListeners();
    this.setupKeyboardControls();
  }

  setupEventListeners() {
    // Reset button
    // Keyboard controls
    // Touch gestures (if needed)
  }

  resetView() {
    // Reset camera to default orbit
  }

  handleKeyboard(e) {
    // Arrow keys → rotate
    // +/- → zoom
    // Space → reset
  }
}

customElements.define('product-3d-viewer-controls', Product3DViewerControls);
```

### UI Components
```
┌─────────────────────────┐
│  [?] Help               │ ← Top-right corner
│                         │
│   3D Model Viewer       │
│   (model-viewer)        │
│                         │
│                    [↺]  │ ← Reset button (bottom-right)
└─────────────────────────┘
```

---

## Related Code Files

### To Create
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhanced-controls.js` (main controls module)

### To Modify
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid` (add control buttons)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhancements.css` (button styles)

### To Reference
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-model.js` (existing pattern)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/global.js` (DeferredMedia pattern)

---

## Implementation Steps

### 1. Create JavaScript Module
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhanced-controls.js`

```javascript
/**
 * Enhanced 3D Viewer Controls
 * Adds keyboard controls, reset button, and accessibility features
 */

class Product3DViewerControls extends HTMLElement {
  constructor() {
    super();
    this.modelViewer = this.querySelector('model-viewer');
    this.resetButton = this.querySelector('[data-3d-reset]');
    this.defaultOrbit = '45deg 75deg 2.5m'; // Default camera position

    if (this.modelViewer) {
      this.init();
    }
  }

  init() {
    this.setupResetButton();
    this.setupKeyboardControls();
    this.setupAccessibility();
    this.saveDefaultOrbit();
  }

  saveDefaultOrbit() {
    // Store initial camera orbit for reset
    this.defaultOrbit = this.modelViewer.getCameraOrbit();
  }

  setupResetButton() {
    if (!this.resetButton) return;

    this.resetButton.addEventListener('click', (e) => {
      e.preventDefault();
      this.resetView();
    });
  }

  resetView() {
    // Reset camera to default position
    this.modelViewer.cameraOrbit = this.defaultOrbit;
    this.modelViewer.fieldOfView = 'auto';

    // Announce to screen readers
    this.announceToScreenReader('View reset to default position');
  }

  setupKeyboardControls() {
    this.modelViewer.setAttribute('tabindex', '0');

    this.modelViewer.addEventListener('keydown', (e) => {
      const rotationSpeed = 5; // degrees per keypress
      const zoomSpeed = 0.1; // meters per keypress

      let handled = false;
      const orbit = this.modelViewer.getCameraOrbit();

      switch(e.key) {
        case 'ArrowLeft':
          orbit.theta -= rotationSpeed * Math.PI / 180;
          handled = true;
          break;
        case 'ArrowRight':
          orbit.theta += rotationSpeed * Math.PI / 180;
          handled = true;
          break;
        case 'ArrowUp':
          orbit.phi -= rotationSpeed * Math.PI / 180;
          handled = true;
          break;
        case 'ArrowDown':
          orbit.phi += rotationSpeed * Math.PI / 180;
          handled = true;
          break;
        case '+':
        case '=':
          orbit.radius -= zoomSpeed;
          handled = true;
          break;
        case '-':
        case '_':
          orbit.radius += zoomSpeed;
          handled = true;
          break;
        case ' ':
          this.resetView();
          handled = true;
          break;
      }

      if (handled) {
        e.preventDefault();
        this.modelViewer.cameraOrbit = `${orbit.theta}rad ${orbit.phi}rad ${orbit.radius}m`;
      }
    });
  }

  setupAccessibility() {
    // Add ARIA labels
    this.modelViewer.setAttribute('aria-label',
      'Interactive 3D model. Use arrow keys to rotate, plus and minus to zoom, space to reset view.'
    );

    // Add focus styles
    this.modelViewer.addEventListener('focus', () => {
      this.modelViewer.classList.add('keyboard-focused');
    });

    this.modelViewer.addEventListener('blur', () => {
      this.modelViewer.classList.remove('keyboard-focused');
    });
  }

  announceToScreenReader(message) {
    const announcement = document.createElement('div');
    announcement.setAttribute('role', 'status');
    announcement.setAttribute('aria-live', 'polite');
    announcement.classList.add('sr-only');
    announcement.textContent = message;
    document.body.appendChild(announcement);

    setTimeout(() => announcement.remove(), 1000);
  }
}

customElements.define('product-3d-viewer-controls', Product3DViewerControls);
```

### 2. Update product-media.liquid
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid`

**Wrap model-viewer with control container:**
```liquid
{%- when 'model' -%}
  <product-3d-viewer-controls class="product-3d-viewer-wrapper">
    <model-viewer
      {%- comment -%} ...existing attributes... {%- endcomment -%}
    >
      {%- comment -%} ...existing slots... {%- endcomment -%}
    </model-viewer>

    {%- comment -%} Reset Button {%- endcomment -%}
    <button
      type="button"
      class="btn-3d-reset"
      data-3d-reset
      aria-label="Reset 3D view to default position">
      <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
        <path d="M4 10C4 6.68629 6.68629 4 10 4C13.3137 4 16 6.68629 16 10C16 13.3137 13.3137 16 10 16"
              stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        <path d="M10 16L8 14M10 16L12 14"
              stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
      </svg>
      <span class="sr-only">Reset view</span>
    </button>

    {%- comment -%} Help Button (Optional) {%- endcomment -%}
    <button
      type="button"
      class="btn-3d-help"
      data-3d-help
      aria-label="Show 3D viewer controls help">
      <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
        <circle cx="10" cy="10" r="8" stroke="currentColor" stroke-width="2"/>
        <path d="M10 14V14.01M10 11C10 9.89543 10.8954 9 12 9C13.1046 9 14 9.89543 14 11C14 12.1046 13.1046 13 12 13H10V14"
              stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
      </svg>
      <span class="sr-only">Help</span>
    </button>
  </product-3d-viewer-controls>
```

### 3. Add Button Styles
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhancements.css`

```css
/* 3D Viewer Controls Wrapper */
.product-3d-viewer-wrapper {
  position: relative;
  display: block;
}

/* Reset Button */
.btn-3d-reset {
  position: absolute;
  bottom: 16px;
  right: 16px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: white;
  border: 1px solid #ddd;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: all 0.2s ease;
  z-index: 10;
}

.btn-3d-reset:hover {
  background: #f5f5f5;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.btn-3d-reset:active {
  transform: scale(0.95);
}

/* Help Button */
.btn-3d-help {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: rgba(255,255,255,0.9);
  border: 1px solid #ddd;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: all 0.2s ease;
  z-index: 10;
}

.btn-3d-help:hover {
  background: white;
}

/* Keyboard Focus Indicator */
model-viewer.keyboard-focused {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

/* Screen Reader Only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0,0,0,0);
  white-space: nowrap;
  border-width: 0;
}

/* Mobile Adjustments */
@media (max-width: 768px) {
  .btn-3d-reset {
    bottom: 12px;
    right: 12px;
    width: 40px;
    height: 40px;
  }

  .btn-3d-help {
    top: 12px;
    right: 12px;
    width: 32px;
    height: 32px;
  }
}
```

### 4. Include JavaScript in Theme
**File:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/layout/theme.liquid`

**Add before `</body>`:**
```liquid
<script src="{{ 'product-3d-viewer-enhanced-controls.js' | asset_url }}" defer></script>
```

### 5. Test Enhanced Controls
```bash
# Test checklist:
# - Click reset button returns to default view
# - Arrow keys rotate model
# - +/- keys zoom in/out
# - Space bar resets view
# - Tab key focuses model-viewer
# - Focus indicator visible on keyboard focus
# - Screen reader announces actions
```

---

## Todo List

- [ ] Create product-3d-viewer-enhanced-controls.js file
- [ ] Implement Product3DViewerControls custom element
- [ ] Add reset button functionality
- [ ] Implement keyboard controls (arrows, +/-, space)
- [ ] Add accessibility features (ARIA, focus, announcements)
- [ ] Update product-media.liquid with control buttons
- [ ] Add reset button SVG icon
- [ ] Add help button (optional)
- [ ] Update CSS with button styles
- [ ] Add keyboard focus indicator styles
- [ ] Include JavaScript in theme.liquid
- [ ] Test reset button on desktop/mobile
- [ ] Test keyboard controls (all keys)
- [ ] Test with screen reader (VoiceOver/NVDA)
- [ ] Verify focus indicator visible
- [ ] Test touch target size (44x44px minimum)
- [ ] Check button z-index doesn't block model
- [ ] Verify buttons don't interfere with gestures

---

## Success Criteria

✅ Reset button returns camera to default position
✅ Arrow keys rotate model smoothly
✅ +/- keys zoom in/out
✅ Space bar resets view
✅ Tab key focuses model-viewer
✅ Keyboard focus indicator visible
✅ Screen reader announces view changes
✅ Buttons are touch-friendly (44x44px)
✅ No JavaScript console errors
✅ Works on desktop and mobile browsers

---

## Risk Assessment

**Low Risk** - Additive features, no breaking changes

**Potential Issues:**
- Keyboard controls conflict with page scroll → Use `preventDefault()` only when focused
- Reset button blocks model interaction → Proper z-index management
- Custom element not supported → Use polyfill or fallback to plain class
- Focus styles not visible → Test with keyboard navigation

**Mitigation:**
- Test keyboard controls with focus states
- Ensure buttons positioned outside main interaction area
- Feature detection for custom elements
- Cross-browser testing (Chrome, Safari, Firefox)

---

## Security Considerations

**XSS Prevention:**
- No user input in JavaScript module
- SVG icons inline (no external sources)
- Event listeners properly bound/unbound

**Accessibility:**
- WCAG 2.2 keyboard navigation compliant
- Screen reader announcements use ARIA live regions
- Focus management prevents keyboard traps

---

## Next Steps

After controls complete:
→ **Phase 05:** Mobile tutorial overlay
→ Test on real mobile devices
→ Gather user feedback on control usability
