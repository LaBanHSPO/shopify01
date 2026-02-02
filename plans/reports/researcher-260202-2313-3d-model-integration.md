# Research Report: 3D Model Integration in Shopify Dawn Theme

**Report Date:** 2026-02-02
**Researcher:** researcher (a72888d)
**Work Context:** /Users/mbpprm/Documents/mybuild/for-sale/shopify01

---

## Executive Summary

Dawn theme has **native 3D model support** via Shopify's model-viewer web component. Integration involves adding GLB/USDZ files through Shopify Admin, which are served from Shopify's CDN and rendered using deferred loading patterns. The existing media gallery infrastructure handles 3D models alongside images/videos with built-in responsive design and AR support.

---

## 1. Shopify Native 3D Model Support

### File Format Requirements
- **Supported formats:** GLB (primary), USDZ (iOS AR)
- **Max file size:** 500MB (auto-optimized if >15MB)
- **Recommended size:** <5MB for optimal performance
- **GLB benefits:** Single file containing model + textures, broad compatibility

### Upload Methods
1. **Shopify Admin UI** (primary): Products → Select product → Upload GLB/USDZ
2. **GraphQL Admin API:**
   - `stagedUploadsCreate` for large files (>15MB)
   - `fileCreate` for smaller assets
   - Files stored once on Shopify CDN, referenced by unique ID across multiple products

### Media Types
Shopify categorizes product media as: `image`, `video`, `external_video`, `model`

---

## 2. Dawn Theme Architecture

### Key Files
```
sections/main-product.liquid          # Main product page template (27k tokens)
snippets/product-media-gallery.liquid # Media gallery container
snippets/product-thumbnail.liquid     # Individual media items
snippets/product-media.liquid         # Media rendering logic
assets/media-gallery.js               # Gallery navigation/switching
assets/product-model.js               # 3D model initialization
assets/global.js                      # DeferredMedia base class
```

### Current Media Handling Flow

**product-media-gallery.liquid:**
- Loops through `product.media` collection
- Detects `first_3d_model` via `| where: 'media_type', 'model' | first`
- Renders XR button if 3D model exists (lines 157-172)
- Uses `product-thumbnail` snippet for each media item

**product-thumbnail.liquid:**
- Wraps media in `<modal-opener>` for lightbox
- Renders preview image for all media types
- For `media_type == 'model'`: wraps in `<product-model>` custom element
- Shows 3D icon badge on thumbnails (lines 274-279)
- Includes `data-shopify-xr` attributes for AR functionality

**product-media.liquid:**
- Conditional rendering based on `media.media_type`
- For models: uses `{{ media | media_tag }}` (line 98)
- Wraps in `<template>` for deferred loading
- Includes XR button with `data-shopify-model3d-id`

**DeferredMedia Pattern:**
```javascript
// assets/global.js (lines 694-726)
class DeferredMedia extends HTMLElement {
  loadContent() {
    // Loads video, model-viewer, or iframe from <template>
    const deferredElement = content.querySelector('video, model-viewer, iframe');
  }
}
```

**ProductModel Component:**
```javascript
// assets/product-model.js
class ProductModel extends DeferredMedia {
  loadContent() {
    Shopify.loadFeatures([{
      name: 'model-viewer-ui',  // Lazy-loads @google/model-viewer v1.21.1
      version: '1.0'
    }]);
  }
}
```

---

## 3. Asset Storage & CDN Strategy

### Current Implementation
- **Shopify CDN (Recommended):** All product media stored on Shopify's CDN (Fastly/Cloudflare)
- **Automatic optimization:** Files >15MB auto-optimized
- **Global distribution:** CDN ensures fast loading worldwide

### Storage Options Comparison

| Method | Pros | Cons | Use Case |
|--------|------|------|----------|
| **Shopify CDN (Media API)** | Native integration, auto-optimization, AR support, unified file system | Plan limits on video/3D count | **PRIMARY - Production use** |
| **Theme Assets** | Version controlled | 500MB limit per asset, no auto-optimization | Dev/testing only |
| **Metafields (file_reference)** | Custom display logic | One file per metafield, no native gallery support | Edge cases only |
| **External CDN** | Unlimited storage | No AR support, breaks Shopify integrations | Not recommended |

**Recommendation:** Use Shopify Media API exclusively for production.

---

## 4. Responsive Layout Patterns

### Desktop Layout Options (via `gallery_layout` setting)
- `thumbnail` - Vertical thumbnail strip + main viewer
- `thumbnail_slider` - Horizontal thumbnail carousel + main viewer
- `stacked` - Stacked full-width images
- `columns` - Two-column grid

### Mobile Layout Options (via `mobile_thumbnails` setting)
- `show` - Show thumbnail strip
- `columns` - Two-column grid
- `hide` - Hide thumbnails (slider only)

### Media Gallery Responsive Behavior
```liquid
{%- assign media_width = 0.65 -%}  {%- comment -%} For "large" media size {%- endcomment -%}
{%- capture sizes -%}
  (min-width: {{ settings.page_width }}px) calc(({{ settings.page_width | minus: 100 | times: media_width | round }} - 4rem) / {{ desktop_columns }}),
  (min-width: 990px) calc({{ media_width | times: 100 }}vw - 4rem),
  (min-width: 750px) calc((100vw - 11.5rem) / 2),
  calc(100vw / {{ mobile_columns }} - 4rem)
{%- endcapture -%}
```

**Breakpoints:**
- `>= 990px`: Desktop layout applies
- `750-989px`: Tablet (2-column if enabled)
- `< 750px`: Mobile (slider or single column)

### 3D Model Specific Considerations
- **Preview images:** All media types render preview images in thumbnails
- **Model-viewer loads on interaction:** Deferred loading prevents blocking page load
- **AR button:** Shows below main viewer when `first_3d_model` exists
- **Mobile AR:** `data-shopify-xr` enables native AR on iOS/Android

---

## 5. Performance Optimization

### Lazy Loading Strategy

**Current Implementation:**
```liquid
{%- render 'product-thumbnail',
  lazy_load: true    {%- comment -%} All media except first featured media {%- endcomment -%}
-%}
```

**Deferred Model Loading:**
1. Initial render: Only preview image + button
2. User clicks: `DeferredMedia.loadContent()` fires
3. `Shopify.loadFeatures()` dynamically injects model-viewer-ui JS (245KB)
4. `<template>` content clones to DOM with `<model-viewer>` element

### Progressive Loading Pattern
```javascript
// media-gallery.js (lines 98-102)
playActiveMedia(activeItem) {
  window.pauseAllMedia();  // Pause all other media first
  const deferredMedia = activeItem.querySelector('.deferred-media');
  if (deferredMedia) deferredMedia.loadContent(false);
}
```

**Key Optimizations:**
- Only active media loads at a time
- Model-viewer JS loads on-demand (not on page load)
- Preview images use responsive srcset with 12 size variants
- CDN serves optimized assets with proper cache headers

### Compression Best Practices
- GLB files: Use Draco compression for geometry
- Texture optimization: Max 2048x2048px, use Basis Universal for WebGL
- Total model size target: 4-5MB uncompressed

---

## 6. Integration Patterns

### Pattern A: Primary 3D Model (Replaces Images)
**Use Case:** Product exists primarily as 3D model

**Implementation:**
1. Upload GLB as first media item (becomes featured media)
2. Add fallback images as additional media
3. Theme automatically renders 3D as primary with image thumbnails

**Backward compatibility:** Images display on non-OS 2.0 themes

### Pattern B: 3D Model + Image Gallery
**Use Case:** Product has photos + supplemental 3D view

**Implementation:**
1. Upload images first (main product photos)
2. Add GLB as additional media item
3. 3D appears in gallery rotation with icon badge
4. AR button appears when model slide is active

**Mobile behavior:** Slider allows swiping between images and 3D

### Pattern C: Variant-Specific 3D Models
**Use Case:** Different variants have different 3D models

**Implementation:**
1. Upload multiple GLB files to product
2. Assign media to specific variants (via `variant_images` pattern)
3. `hide_variants` setting controls visibility

**Current limitation:** Dawn's `variant_images` mainly handles images; 3D variant association needs custom logic

---

## 7. Backward Compatibility

### Products Without 3D Models
**Automatic Fallback:** Theme checks `if first_3d_model` before rendering XR button
```liquid
{%- if first_3d_model -%}
  <button data-shopify-xr>View in your space</button>
{%- endif -%}
```

**Gallery Behavior:**
- Mixed media (images + videos + 3D) render seamlessly
- Media count includes all types: `product.media.size`
- No 3D models = standard image gallery

### Theme Version Requirements
- **OS 2.0 themes:** Full 3D support (Dawn, Refresh, etc.)
- **OS 1.0 themes:** No native 3D support (requires custom development)
- **Legacy themes:** May not support media types beyond images

### Browser Compatibility
- **model-viewer:** WebGL support required (all modern browsers)
- **AR (iOS):** QuickLook USDZ support (iOS 12+)
- **AR (Android):** Scene Viewer GLB support (ARCore devices)
- **Fallback:** Preview image shows on unsupported browsers

---

## 8. Implementation Guidance

### Minimal Integration Steps

**Step 1:** Upload GLB via Shopify Admin
```
Products → [Select Product] → Media → Add media → Upload GLB
```

**Step 2:** Verify Dawn theme renders correctly
- Check product page shows 3D icon on thumbnail
- Click thumbnail to load model-viewer
- Verify AR button appears (if `first_3d_model` exists)

**Step 3:** Test responsive behavior
- Desktop: Verify gallery layout
- Mobile: Test slider, AR button placement
- Tablet: Verify 2-column behavior

### Custom Integration (If Needed)

**Scenario:** Display 3D as primary on product page, replace image gallery

**Modifications Required:**
1. **main-product.liquid:** Add conditional logic to prioritize 3D
2. **product-media-gallery.liquid:** Modify featured media selection
3. **CSS:** Adjust viewer dimensions for prominence

**Code Example:**
```liquid
{%- assign first_3d_model = product.media | where: 'media_type', 'model' | first -%}
{%- if first_3d_model -%}
  {%- comment -%} Render 3D as hero, images as thumbnails {%- endcomment -%}
  {% render 'product-3d-hero', media: first_3d_model %}
{%- else -%}
  {%- comment -%} Standard image gallery {%- endcomment -%}
  {% render 'product-media-gallery' %}
{%- endif -%}
```

### Performance Checklist
- [ ] GLB file <5MB
- [ ] Draco compression applied
- [ ] Textures ≤2048px
- [ ] Preview images optimized (WebP if possible)
- [ ] Test on 3G connection
- [ ] Verify model-viewer loads only on interaction

---

## 9. Key Insights

### Strengths of Current Architecture
1. **Zero custom code required:** Upload GLB → automatic rendering
2. **Deferred loading built-in:** Model-viewer JS loads on-demand
3. **CDN-first approach:** Shopify handles optimization/distribution
4. **AR support included:** iOS/Android AR works automatically
5. **Responsive by default:** Mobile/desktop layouts handled

### Limitations
1. **Plan-based limits:** Video/3D model count varies by Shopify plan
2. **Large JS payload:** model-viewer is 245KB (mitigated by lazy load)
3. **No variant-specific 3D logic:** Requires custom development
4. **Grid layout default:** Dawn uses grid vs traditional slider (can customize)

### Best Practices
- Store all 3D models via Shopify Media API (not theme assets)
- Use GLB format (not loose GLTF + textures)
- Always include fallback images
- Test AR on physical iOS/Android devices
- Monitor Lighthouse performance scores
- Use `lazy_load: true` for all media except featured

---

## 10. Next Steps

### Immediate Actions
1. **Audit existing products:** Identify candidates for 3D models
2. **GLB file preparation:** Compress models to <5MB
3. **Upload test model:** Verify Dawn theme rendering
4. **Mobile testing:** Test AR functionality on devices

### Custom Development Needs
- Variant-specific 3D model switching (if required)
- Custom hero 3D viewer layout (if replacing gallery)
- Progressive enhancement for older browsers
- Analytics tracking for 3D interaction events

### Documentation to Review
- [Shopify 3D Models Guide](https://help.shopify.com/en/partners/manage-clients-stores/creating-media/3d-models/creating-3d-models)
- [Product Media API](https://shopify.dev/docs/apps/build/online-store/product-media)
- [ModelViewer Component Docs](https://shopify.dev/api/hydrogen/components/primitive/modelviewer)
- [Performance Best Practices](https://shopify.dev/docs/storefronts/themes/best-practices/performance)

---

## Unresolved Questions

1. **Variant-specific 3D models:** Does client need different 3D models per variant, or single model per product?
2. **Gallery vs Hero layout:** Should 3D replace image gallery entirely, or integrate within existing gallery?
3. **Fallback strategy:** For products without 3D models, should there be visual indicators encouraging 3D upload?
4. **Analytics requirements:** Need to track 3D interaction metrics (loads, AR sessions)?
5. **Custom viewer controls:** Are default model-viewer controls sufficient, or need custom UI?
6. **Internationalization:** Do 3D models need localization (e.g., different textures per market)?

---

## Sources

- [Add a 3D Model to Shopify Products (GLB/USDZ) – Easy Guide](https://www.3dimages.ai/blog/integrate-3d-model-shopify)
- [How to add 3D models to Shopify? – How Commerce](https://www.howcommerce.com/shopify-3d-models/)
- [Shopify Help Center | Product media types](https://help.shopify.com/en/manual/products/product-media/product-media-types)
- [Manage media for products and collections](https://shopify.dev/docs/apps/build/online-store/product-media)
- [Media - GraphQL Admin](https://shopify.dev/docs/api/admin-graphql/latest/interfaces/media)
- [Mastering Shopify CDN Optimization for Superior Ecommerce Performance | Praella](https://praella.com/blogs/shopify-insights/mastering-shopify-cdn-optimization-for-superior-ecommerce-performance)
- [Performance best practices for Shopify themes](https://shopify.dev/docs/storefronts/themes/best-practices/performance)
- [ModelViewer - Shopify.dev](https://shopify.dev/api/hydrogen/components/primitive/modelviewer)
- [How to Embed 3D Models on the Web – Cloud Four](https://cloudfour.com/thinks/how-to-embed-3d-models-on-the-web/)
- [Shopify Dawn Theme Review: The Next Generation Of Shopify Theme?](https://pagefly.io/blogs/shopify/shopify-dawn-theme)
- [Product Image Slider for Shopify Dawn Theme Using Web Components](https://barstool.engineering/product-image-slider-for-shopify-dawn-theme/)
- [Shopify Help Center | Product media](https://help.shopify.com/en/manual/products/product-media)
- [Shopify Help Center | Creating 3D models for merchants](https://help.shopify.com/en/partners/manage-clients-stores/creating-media/3d-models/creating-3d-models)

---

**Report Complete**
File: `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-3d-model-integration.md`
