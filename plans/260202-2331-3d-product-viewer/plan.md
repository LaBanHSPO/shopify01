---
title: "3D Product Viewer with Zoom & Rotation for Shopify Dawn"
description: "Add mobile-first 3D viewer using model-viewer, zoom/rotate controls, tutorial overlay"
status: pending
priority: P1
effort: 8-12h
branch: main
tags: [3d-viewer, model-viewer, mobile-ux, shopify, dawn-theme]
created: 2026-02-02
---

# 3D Product Viewer Implementation Plan

**Objective:** Add 3D product viewer with zoom + hand rotation to Shopify Dawn theme using native model-viewer support.

**Approach:** Leverage Dawn's built-in 3D infrastructure, enhance with custom zoom controls, mobile gestures, tutorial overlay.

**Tech Stack:** Google model-viewer (native to Dawn), GLB format, Shopify Media API, minimal JavaScript for enhancements.

---

## Context

**Research Reports:**
- [3D Model Viewer Comparison](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-3d-model-viewer-comparison.md)
- [3D Model Integration in Dawn](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-3d-model-integration.md)
- [Mobile 3D Viewer UX](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-mobile-3d-viewer-ux.md)

**Performance Targets:**
- <5MB GLB files, <50k polygons
- 30+ FPS mobile, <3s load time
- 1-finger drag = rotate, 2-finger pinch = zoom

---

## Phase Overview

| Phase | Description | Status | Effort |
|-------|-------------|--------|--------|
| [01](phase-01-setup-analysis.md) | Setup & Analysis | ⏳ Pending | 1h |
| [02](phase-02-asset-preparation.md) | 3D Asset Preparation | ⏳ Pending | 2h |
| [03](phase-03-product-media-integration.md) | Product Media Integration | ⏳ Pending | 2h |
| [04](phase-04-enhanced-controls.md) | Enhanced Controls | ⏳ Pending | 2h |
| [05](phase-05-mobile-optimization.md) | Mobile Optimization | ⏳ Pending | 2h |
| [06](phase-06-testing-refinement.md) | Testing & Refinement | ⏳ Pending | 2h |
| [07](phase-07-documentation.md) | Documentation | ⏳ Pending | 1h |

**Total Estimated Effort:** 12h

---

## Key Decisions

**Library:** Google model-viewer (native Dawn support, zero config)
**Format:** GLB (single file, broad compatibility)
**Storage:** Shopify Media API (CDN-optimized)
**Layout:** 3D-first with image gallery fallback
**Mobile:** Touch gestures + tutorial overlay (first visit)

---

## Success Criteria

✅ 3D model displays as primary product media
✅ Zoom + rotation work via touch/mouse
✅ Tutorial overlay on first visit (mobile)
✅ 30+ FPS on iPhone 12/Galaxy A52
✅ <3s load time for full quality
✅ Keyboard accessible (WCAG 2.2)
✅ Graceful fallback to 2D images on error

---

## Risk Mitigation

**Large file sizes:** Enforce <5MB limit, optimize before upload
**Browser compatibility:** Test on iOS Safari 15.4+, Android Chrome 94+
**Performance:** Progressive loading (low-poly → full quality)
**User confusion:** Tutorial overlay with gesture hints

---

## Next Steps After Completion

- Analytics tracking (3D interaction events)
- AR mode enhancement (iOS Quick Look, Android Scene Viewer)
- Variant-specific 3D models
- Hotspot annotations for product features
