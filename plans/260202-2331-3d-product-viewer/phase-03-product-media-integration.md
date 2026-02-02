# Phase 03: Product Media Integration

**Priority:** P1
**Status:** Pending
**Effort:** 2h
**Dependencies:** Phase 01, 02 complete

---

## Context Links

- [Phase 01 Analysis](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/phase-01-setup-analysis.md)
- [Research: Dawn Integration](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-3d-model-integration.md)

---

## Overview

Upload optimized GLB to Shopify, modify product-media.liquid to prioritize 3D models, configure model-viewer attributes, ensure responsive layout.

---

## Key Insights

- Dawn auto-generates `<model-viewer>` for uploaded GLB files
- `product.media | where: 'media_type', 'model'` filters 3D models
- Model-viewer attributes control behavior (camera-controls, auto-rotate)
- Deferred loading via `<template>` prevents blocking
- XR button appears automatically when `first_3d_model` exists

---

## Requirements

### Functional
- [ ] GLB files uploaded to Shopify product media
- [ ] 3D model appears as primary product media
- [ ] Model-viewer configured with zoom + rotation
- [ ] Responsive layout (desktop 600px, mobile 400px height)
- [ ] Fallback to image gallery if no 3D model

### Non-Functional
- [ ] No breaking changes to existing image gallery
- [ ] Backward compatible with products without 3D
- [ ] Performance: lazy loading, poster images
- [ ] Accessibility: ARIA labels, keyboard controls

---

## Architecture

### Modified Files
```
snippets/product-media.liquid        # Add custom model-viewer attributes
snippets/product-media-gallery.liquid # Prioritize 3D model display
sections/main-product.liquid         # Optional: layout adjustments
assets/product-model.js              # Optional: enhance controls (Phase 04)
```

### Model-Viewer Configuration
```html
<model-viewer
  src="{{ media | model_viewer_url }}"
  ios-src="{{ media | model_viewer_url: format: 'usdz' }}"
  poster="{{ media.preview_image | img_url: 'master' }}"
  alt="{{ media.alt | escape }}"
  camera-controls              <!-- Enable zoom + rotate -->
  touch-action="pan-y"         <!-- Allow page scroll -->
  auto-rotate                  <!-- Subtle rotation when idle -->
  auto-rotate-delay="3000"     <!-- Start after 3s -->
  camera-orbit="45deg 75deg 2.5m" <!-- Default view angle -->
  min-camera-orbit="auto auto 1m"  <!-- Min zoom -->
  max-camera-orbit="auto auto 5m"  <!-- Max zoom -->
  loading="lazy"               <!-- Load on interaction -->
  reveal="interaction"         <!-- Show after user clicks -->
  exposure="1.0"               <!-- Lighting intensity -->
  shadow-intensity="1">        <!-- Ground shadow -->

  <div slot="progress-bar" class="model-viewer-progress">
    <div class="progress-bar-update"></div>
  </div>
</model-viewer>
```

---

## Related Code Files

### To Modify
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid` (add custom attributes)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media-gallery.liquid` (prioritize 3D)

### To Review (Context)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/sections/main-product.liquid`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-model.js`

### To Create
- CSS file: `assets/product-3d-viewer-enhancements.css` (custom styling)

---

## Implementation Steps

### 1. Upload GLB to Shopify
```bash
# Via Shopify Admin UI:
# 1. Go to Products → Select product
# 2. Click "Add media" → Upload GLB file
# 3. Verify preview image generated
# 4. Check USDZ auto-generated for iOS
# 5. Set media order (3D first, then images)
```

**Alternative: GraphQL API** (if bulk upload needed)
```graphql
mutation {
  stagedUploadsCreate(input: {
    resource: PRODUCT_MEDIA
    filename: "product-model.glb"
    mimeType: "model/gltf-binary"
    fileSize: "4200000"
  }) {
    stagedTargets {
      url
      parameters {
        name
        value
      }
    }
  }
}
```

### 2. Modify product-media.liquid
**Location:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid`

**Find:** Existing model-viewer rendering (around line 98)
```liquid
{%- when 'model' -%}
  {{ media | media_tag }}
```

**Replace with:** Enhanced model-viewer configuration
```liquid
{%- when 'model' -%}
  <model-viewer
    src="{{ media | model_viewer_url }}"
    ios-src="{{ media | model_viewer_url: format: 'usdz' }}"
    poster="{{ media.preview_image | img_url: 'master' }}"
    alt="{{ media.alt | default: product.title | escape }}"
    camera-controls
    touch-action="pan-y"
    auto-rotate
    auto-rotate-delay="3000"
    rotation-per-second="30deg"
    camera-orbit="45deg 75deg 2.5m"
    min-camera-orbit="auto auto 1m"
    max-camera-orbit="auto auto 5m"
    loading="lazy"
    reveal="interaction"
    exposure="1.0"
    shadow-intensity="1"
    class="product-3d-viewer"
    data-shopify-model3d-id="{{ media.id }}">

    {%- comment -%} Progress bar slot {%- endcomment -%}
    <div slot="progress-bar" class="model-viewer-progress">
      <div class="progress-bar-update"></div>
    </div>

    {%- comment -%} AR button (if supported) {%- endcomment -%}
    <button slot="ar-button" class="model-viewer-ar-button">
      {% render 'icon-3d' %} View in your space
    </button>
  </model-viewer>
```

### 3. Add CSS Styling
**Create:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-3d-viewer-enhancements.css`

```css
/* 3D Viewer Container */
.product-3d-viewer {
  width: 100%;
  height: 600px;
  background-color: #f9f9f9;
  border-radius: 8px;
  overflow: hidden;
}

/* Progress Bar */
.model-viewer-progress {
  width: 100%;
  height: 4px;
  background: rgba(0, 0, 0, 0.1);
  position: absolute;
  bottom: 0;
  left: 0;
}

.progress-bar-update {
  height: 100%;
  background: #000;
  transition: width 0.3s ease;
}

/* AR Button */
.model-viewer-ar-button {
  position: absolute;
  bottom: 16px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 24px;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.model-viewer-ar-button:hover {
  background: #f5f5f5;
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .product-3d-viewer {
    height: 400px;
  }
}

@media (max-width: 480px) {
  .product-3d-viewer {
    height: 350px;
  }
}
```

### 4. Include CSS in Theme
**Modify:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/layout/theme.liquid`

**Add before `</head>`:**
```liquid
{{ 'product-3d-viewer-enhancements.css' | asset_url | stylesheet_tag }}
```

### 5. Prioritize 3D Model in Gallery
**Modify:** `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media-gallery.liquid`

**Find:** Featured media logic (around line 20-30)
```liquid
{%- assign featured_media = product.selected_or_first_available_variant.featured_media | default: product.featured_media -%}
```

**Enhance to prioritize 3D:**
```liquid
{%- assign first_3d_model = product.media | where: 'media_type', 'model' | first -%}
{%- if first_3d_model -%}
  {%- assign featured_media = first_3d_model -%}
{%- else -%}
  {%- assign featured_media = product.selected_or_first_available_variant.featured_media | default: product.featured_media -%}
{%- endif -%}
```

### 6. Test on Development Store
```bash
# Start Shopify CLI dev server
shopify theme dev --store your-dev-store.myshopify.com

# Test checklist:
# - 3D model appears as primary media
# - Click thumbnail to load model-viewer
# - Drag to rotate, pinch to zoom
# - Verify poster image shows during load
# - Check AR button appears (mobile)
# - Test fallback (products without 3D)
```

### 7. Verify Responsive Layout
```bash
# Test breakpoints:
# Desktop (>990px): 600px height
# Tablet (750-989px): 500px height
# Mobile (<750px): 400px height

# Use browser dev tools:
# iPhone 12: 390x844
# Galaxy A52: 360x800
# iPad: 768x1024
```

---

## Todo List

- [ ] Upload optimized GLB files to Shopify product media
- [ ] Verify USDZ files generated automatically
- [ ] Set 3D model as first media item (drag in admin)
- [ ] Read current product-media.liquid implementation
- [ ] Modify product-media.liquid with enhanced model-viewer attributes
- [ ] Create product-3d-viewer-enhancements.css file
- [ ] Add CSS file to theme.liquid head
- [ ] Modify product-media-gallery.liquid to prioritize 3D
- [ ] Test model loads on product page
- [ ] Verify camera-controls (zoom/rotate) work
- [ ] Test auto-rotate after 3s idle
- [ ] Check poster image displays during load
- [ ] Verify AR button appears on mobile
- [ ] Test fallback for products without 3D models
- [ ] Test responsive layouts (desktop, tablet, mobile)
- [ ] Verify no JavaScript console errors
- [ ] Check accessibility (keyboard controls, ARIA)

---

## Success Criteria

✅ GLB uploaded and visible in Shopify admin
✅ 3D model displays as primary product media
✅ Zoom and rotation work via mouse/touch
✅ Auto-rotate starts after 3s idle
✅ Poster image shows during loading
✅ AR button visible on supported devices
✅ Responsive: 600px desktop, 400px mobile
✅ No breaking changes to image gallery
✅ Products without 3D fall back to images gracefully
✅ Keyboard accessible (arrow keys, +/- zoom)

---

## Risk Assessment

**Medium Risk** - Modifying core Liquid templates

**Potential Issues:**
- Breaking existing image gallery → Test thoroughly with/without 3D
- Model-viewer not loading → Verify CDN URL, check console errors
- Layout shifts → Test responsive breakpoints
- XR button not appearing → Check `data-shopify-model3d-id` attribute

**Mitigation:**
- Create theme backup before changes
- Test on development store first
- Use Git version control (commit after each change)
- Keep original code commented out for rollback
- Test with multiple product types (with/without 3D)

---

## Security Considerations

**Shopify Media API:**
- Files served from Shopify CDN (HTTPS enforced)
- CORS headers handled automatically
- File size limits enforced (15MB max)

**XSS Prevention:**
- Use Liquid escape filter: `{{ media.alt | escape }}`
- Sanitize user-generated content in alt text
- No direct HTML injection in model-viewer attributes

---

## Next Steps

After integration complete:
→ **Phase 04:** Add custom zoom controls UI (optional)
→ **Phase 05:** Mobile tutorial overlay
→ Test on real devices (iOS/Android)
