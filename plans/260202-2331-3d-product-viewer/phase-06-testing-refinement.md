# Phase 06: Testing & Refinement

**Priority:** P1
**Status:** Pending
**Effort:** 2h
**Dependencies:** Phase 03, 04, 05 complete

---

## Context Links

- [Phase 05 Mobile Optimization](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/phase-05-mobile-optimization.md)
- [Research: Mobile UX](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-mobile-3d-viewer-ux.md)

---

## Overview

Comprehensive cross-browser testing, mobile device testing, performance profiling, accessibility audit, bug fixes, refinement based on findings.

---

## Key Insights

- Test on real devices, not just simulators
- Target: iPhone 12/SE, Galaxy A52, budget Android
- Lighthouse score target: 90+ performance
- WCAG 2.2 Level AA compliance required
- 30+ FPS on mobile is non-negotiable

---

## Requirements

### Functional Testing
- [ ] 3D model loads on all target browsers
- [ ] Zoom/rotation work via mouse, touch, keyboard
- [ ] Tutorial shows correctly on mobile
- [ ] Reset button functions properly
- [ ] AR button appears on supported devices
- [ ] Fallback to images when 3D fails

### Performance Testing
- [ ] Load time <3s on Fast 3G
- [ ] 30+ FPS during rotation (mobile)
- [ ] No memory leaks (long sessions)
- [ ] Lighthouse score 90+ (performance)
- [ ] WebPageTest budget met

### Accessibility Testing
- [ ] WCAG 2.2 Level AA compliance
- [ ] Screen reader compatible
- [ ] Keyboard navigation complete
- [ ] Focus indicators visible
- [ ] Color contrast sufficient

### Cross-Browser Testing
- [ ] Chrome 94+ (desktop/mobile)
- [ ] Safari 15.4+ (desktop/iOS)
- [ ] Firefox 63+ (desktop/mobile)
- [ ] Edge 79+ (desktop)
- [ ] Samsung Internet (Android)

---

## Architecture

### Testing Matrix
```
Device Types:
├── Desktop (Windows, macOS)
├── Mobile (iOS, Android)
└── Tablet (iPad, Android tablet)

Browsers:
├── Chrome/Chromium (94+)
├── Safari (15.4+)
├── Firefox (63+)
├── Edge (79+)
└── Samsung Internet

Test Scenarios:
├── Happy path (3D loads normally)
├── Slow network (Fast 3G)
├── Error cases (invalid GLB)
├── Fallback (no WebGL support)
└── Accessibility (keyboard, screen reader)
```

---

## Related Code Files

### To Test
- All modified Liquid files
- All JavaScript modules
- All CSS files
- Shopify product pages with 3D models

### To Create
- Test report: `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/reports/phase-06-testing-report.md`
- Bug log: `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/reports/phase-06-bug-log.md`

---

## Implementation Steps

### 1. Desktop Browser Testing

**Chrome (Desktop)**
```bash
# Test checklist:
# - Open product page with 3D model
# - Verify model loads within 3s
# - Drag to rotate (smooth, no lag)
# - Scroll wheel to zoom
# - Click reset button
# - Test keyboard controls (arrows, +/-, space)
# - Open DevTools Console (check for errors)
# - Network panel (verify GLB file size, load time)
```

**Safari (Desktop)**
```bash
# macOS Safari specific tests:
# - Verify model-viewer loads (WebGL support)
# - Test trackpad gestures (pinch-to-zoom)
# - Check for rendering quirks (known Safari issues)
# - Verify USDZ file generated for AR
# - Test with Safari's Responsive Design Mode
```

**Firefox (Desktop)**
```bash
# Firefox specific tests:
# - Verify model-viewer polyfill loads
# - Test mouse controls
# - Check memory usage (about:performance)
# - Verify no console warnings
```

**Edge (Desktop)**
```bash
# Edge Chromium tests:
# - Same as Chrome (Chromium-based)
# - Verify no compatibility issues
```

### 2. Mobile Device Testing

**iOS Testing (Real Device)**
```bash
# iPhone 12 Pro (iOS 17+)
# - Connect via USB, open Safari
# - Test 3D model loads
# - One-finger drag to rotate
# - Two-finger pinch to zoom
# - Verify tutorial shows on first visit
# - Test AR mode (Quick Look)
# - Check battery drain during 5min session
# - Monitor FPS (Xcode Instruments)

# iPhone SE 2020 (Budget device)
# - Same tests as iPhone 12
# - Verify 30+ FPS maintained
# - Check for thermal throttling
```

**Android Testing (Real Device)**
```bash
# Galaxy A52 (Android 13+)
# - Open Chrome browser
# - Test 3D model loads
# - Touch gestures (drag, pinch)
# - Verify tutorial overlay
# - Test AR mode (Scene Viewer)
# - Monitor FPS (Chrome DevTools remote debug)

# Budget Android (2-3 years old)
# - Same tests as Galaxy A52
# - Monitor for lag/stuttering
# - Verify graceful degradation
```

### 3. Performance Profiling

**Lighthouse Audit**
```bash
# Run Lighthouse in Chrome DevTools:
# 1. Open product page with 3D model
# 2. DevTools > Lighthouse
# 3. Select "Performance" + "Accessibility"
# 4. Run audit

# Target scores:
# - Performance: 90+
# - Accessibility: 95+
# - Best Practices: 90+
# - SEO: 90+

# Key metrics:
# - First Contentful Paint: <1.8s
# - Largest Contentful Paint: <2.5s
# - Total Blocking Time: <200ms
# - Cumulative Layout Shift: <0.1
```

**WebPageTest**
```bash
# Run WebPageTest (webpagetest.org):
# 1. Enter product page URL
# 2. Select "Moto G4 - 3G" (mobile simulation)
# 3. Run test

# Analyze:
# - Load time breakdown
# - Waterfall chart (GLB file size/timing)
# - Filmstrip view (visual progress)
# - Performance budget compliance
```

**Chrome DevTools Performance Panel**
```bash
# Record performance profile:
# 1. Open DevTools > Performance
# 2. Start recording
# 3. Interact with 3D model (rotate, zoom)
# 4. Stop after 10 seconds

# Analyze:
# - FPS (should be 30-60)
# - Main thread activity
# - Memory allocation
# - Layout shifts
```

### 4. Accessibility Audit

**Keyboard Navigation**
```bash
# Test without mouse:
# - Tab to focus model-viewer
# - Arrow keys to rotate
# - +/- keys to zoom
# - Space to reset
# - Esc to dismiss tutorial (mobile)
# - Verify focus indicators visible
# - No keyboard traps
```

**Screen Reader Testing**
```bash
# VoiceOver (macOS/iOS):
# - Enable VoiceOver (Cmd+F5)
# - Navigate to 3D viewer
# - Verify ARIA labels announced
# - Test control button labels
# - Verify actions announced

# NVDA (Windows):
# - Enable NVDA
# - Navigate to product page
# - Test same as VoiceOver
```

**WAVE Browser Extension**
```bash
# Install WAVE extension
# Run audit on product page
# Check for:
# - Missing alt text
# - Insufficient color contrast
# - Missing ARIA labels
# - Heading structure issues
```

### 5. Error Handling & Fallback Testing

**Invalid GLB File**
```bash
# Test with corrupted/missing GLB:
# - Remove GLB file from Shopify
# - Verify fallback image appears
# - Check console error handling
# - Verify no JavaScript crashes
```

**Slow Network**
```bash
# Chrome DevTools Network throttling:
# - Set to "Fast 3G"
# - Reload product page
# - Verify poster image shows immediately
# - Check progressive loading (low-poly → full)
# - Verify load time <3s
```

**WebGL Not Supported**
```bash
# Test in browser without WebGL:
# - Disable WebGL (chrome://flags)
# - Verify fallback to static image
# - Check for graceful error message
```

### 6. Bug Fixing & Refinement

**Create Bug Log**
```markdown
# Bug #1: Rotation lag on Galaxy A52
- Severity: Medium
- Description: FPS drops to 20 during rotation
- Reproduction: Drag quickly on Galaxy A52
- Fix: Reduce polygon count from 60k to 45k
- Status: Fixed

# Bug #2: Tutorial doesn't dismiss on iOS Safari
- Severity: High
- Description: Tap on "Got it" button unresponsive
- Reproduction: iPhone 12, Safari 16
- Fix: Add -webkit-tap-highlight-color CSS
- Status: Fixed
```

**Prioritize Fixes**
```
P1 (Critical): Blocks core functionality
P2 (High): Degrades user experience
P3 (Medium): Minor issues, edge cases
P4 (Low): Nice-to-have improvements
```

### 7. Regression Testing

**After Each Fix**
```bash
# Re-run tests that failed
# Verify fix doesn't break other features
# Update bug log with status
# Commit fix with descriptive message
```

---

## Todo List

### Desktop Testing
- [ ] Test Chrome (Windows/macOS)
- [ ] Test Safari (macOS)
- [ ] Test Firefox (Windows/macOS)
- [ ] Test Edge (Windows)
- [ ] Verify all browsers support model-viewer
- [ ] Check console for errors (all browsers)

### Mobile Testing
- [ ] Test iOS Safari (iPhone 12)
- [ ] Test iOS Safari (iPhone SE)
- [ ] Test Android Chrome (Galaxy A52)
- [ ] Test Android Chrome (budget device)
- [ ] Test Samsung Internet (Android)
- [ ] Verify tutorial shows on mobile only
- [ ] Test AR mode (iOS Quick Look)
- [ ] Test AR mode (Android Scene Viewer)

### Performance Testing
- [ ] Run Lighthouse audit (desktop)
- [ ] Run Lighthouse audit (mobile)
- [ ] Run WebPageTest (3G simulation)
- [ ] Record Chrome Performance profile
- [ ] Measure FPS during rotation (30+ target)
- [ ] Check memory usage (no leaks)
- [ ] Verify load time <3s (Fast 3G)
- [ ] Check GLB file size <5MB

### Accessibility Testing
- [ ] Test keyboard navigation (all controls)
- [ ] Test VoiceOver (macOS/iOS)
- [ ] Test NVDA (Windows)
- [ ] Run WAVE extension audit
- [ ] Verify focus indicators visible
- [ ] Check color contrast (WCAG AA)
- [ ] Test with screen magnification
- [ ] Verify ARIA labels correct

### Error Handling Testing
- [ ] Test with missing GLB file
- [ ] Test with corrupted GLB file
- [ ] Test slow network (Fast 3G)
- [ ] Test WebGL disabled
- [ ] Test localStorage disabled
- [ ] Verify fallback to images
- [ ] Check error messages user-friendly

### Bug Fixing
- [ ] Document all bugs found
- [ ] Prioritize bugs (P1-P4)
- [ ] Fix critical bugs (P1)
- [ ] Fix high-priority bugs (P2)
- [ ] Retest after each fix
- [ ] Update bug log with status

### Documentation
- [ ] Create testing report with findings
- [ ] Document browser compatibility matrix
- [ ] List known issues (if any)
- [ ] Update bug log with all issues
- [ ] Create performance benchmark report

---

## Success Criteria

✅ All target browsers load 3D model correctly
✅ 30+ FPS on iPhone 12/Galaxy A52
✅ Load time <3s on Fast 3G
✅ Lighthouse performance score 90+
✅ WCAG 2.2 Level AA compliant
✅ Keyboard navigation fully functional
✅ Screen reader announces all controls
✅ Tutorial shows on mobile (first visit)
✅ AR mode works on iOS/Android
✅ Fallback to images on error
✅ No console errors on any browser
✅ All P1/P2 bugs fixed
✅ Testing report completed

---

## Risk Assessment

**High Risk** - Real-world devices may reveal critical issues

**Potential Issues:**
- Low FPS on budget devices → Reduce polygon count
- Browser-specific rendering bugs → Add polyfills/workarounds
- Accessibility violations → Fix before launch
- Memory leaks during long sessions → Proper cleanup
- Tutorial doesn't show on some devices → localStorage fallback

**Mitigation:**
- Test on variety of devices (new + old)
- Allocate time for bug fixes
- Keep performance budget strict
- Use progressive enhancement
- Have rollback plan if critical issues found

---

## Security Considerations

**Testing Data:**
- Use non-sensitive test products
- Don't expose internal URLs publicly
- Sanitize screenshots before sharing

**Performance Monitoring:**
- No tracking without user consent
- Monitor for DDoS during load testing
- Rate limit API calls

---

## Next Steps

After testing complete:
→ **Phase 07:** Documentation
→ Deploy to staging for final QA
→ Client review and approval
→ Production deployment
