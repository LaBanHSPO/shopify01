# Phase 01: Setup & Analysis

**Priority:** P1
**Status:** Pending
**Effort:** 1h
**Dependencies:** None

---

## Context Links

- [Research: Dawn 3D Integration](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/reports/researcher-260202-2313-3d-model-integration.md)
- [Dawn Theme README](/Users/mbpprm/Documents/mybuild/for-sale/shopify01/README.md)

---

## Overview

Analyze Dawn's existing 3D model support, identify files to modify, verify model-viewer version, document current media gallery architecture.

---

## Key Insights

- Dawn has **native model-viewer support** via `assets/product-model.js`
- Model-viewer loads on-demand (deferred loading pattern)
- GLB/USDZ handled automatically via Shopify Media API
- Current implementation: 245KB model-viewer-ui v1.21.1
- Deferred loading prevents page load blocking

---

## Requirements

### Functional
- [ ] Identify all files related to product media rendering
- [ ] Document current model-viewer configuration
- [ ] Map out media gallery flow (Liquid + JS)
- [ ] Verify CDN paths for model-viewer library

### Non-Functional
- [ ] No changes to existing functionality during analysis
- [ ] Document findings in structured format
- [ ] Create modification checklist for next phases

---

## Architecture

### Current Flow
```
main-product.liquid
  └─> product-media-gallery.liquid
      └─> product-thumbnail.liquid
          └─> product-media.liquid
              └─> DeferredMedia (global.js)
                  └─> ProductModel (product-model.js)
```

### Key Files to Analyze
```
sections/main-product.liquid          # Product page template
snippets/product-media-gallery.liquid # Gallery container
snippets/product-thumbnail.liquid     # Thumbnail items
snippets/product-media.liquid         # Media rendering
assets/media-gallery.js               # Gallery navigation
assets/product-model.js               # Model initialization
assets/global.js                      # DeferredMedia class
```

---

## Related Code Files

### To Analyze (Read-Only)
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/sections/main-product.liquid`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media-gallery.liquid`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-thumbnail.liquid`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/snippets/product-media.liquid`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/product-model.js`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/media-gallery.js`
- `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/assets/global.js` (DeferredMedia class)

### Documentation
- Create analysis report: `/Users/mbpprm/Documents/mybuild/for-sale/shopify01/plans/260202-2331-3d-product-viewer/reports/phase-01-analysis-report.md`

---

## Implementation Steps

### 1. Read Current Implementation Files
```bash
# Read product page template
Read sections/main-product.liquid (first 500 lines to understand structure)

# Read media gallery components
Read snippets/product-media-gallery.liquid
Read snippets/product-thumbnail.liquid
Read snippets/product-media.liquid

# Read JavaScript components
Read assets/product-model.js
Read assets/media-gallery.js
Read assets/global.js (focus on DeferredMedia class)
```

### 2. Document Model-Viewer Configuration
- [ ] Current version of model-viewer library
- [ ] CDN URL being used
- [ ] Default attributes set on `<model-viewer>`
- [ ] Loading strategy (lazy, eager, interaction)

### 3. Map Media Gallery Logic
- [ ] How gallery detects 3D models (`first_3d_model` filter)
- [ ] Thumbnail rendering for model media type
- [ ] XR button positioning and visibility logic
- [ ] Deferred loading trigger mechanism

### 4. Identify Modification Points
- [ ] Where to add custom zoom controls
- [ ] Where to inject tutorial overlay HTML
- [ ] JavaScript files for gesture enhancement
- [ ] CSS files for custom styling

### 5. Create Analysis Report
Document findings:
- Current architecture diagram
- Model-viewer attributes used
- Liquid variables available
- JavaScript extension points
- CSS class structure

---

## Todo List

- [ ] Read main-product.liquid structure
- [ ] Read product-media-gallery.liquid logic
- [ ] Read product-thumbnail.liquid rendering
- [ ] Read product-media.liquid conditional logic
- [ ] Read product-model.js DeferredMedia extension
- [ ] Read media-gallery.js navigation flow
- [ ] Read global.js DeferredMedia base class
- [ ] Document model-viewer version and CDN URL
- [ ] Map Liquid variable flow for 3D models
- [ ] Identify where XR button is rendered
- [ ] Document gallery layout options (thumbnail, slider, stacked)
- [ ] Create modification points checklist
- [ ] Write phase-01-analysis-report.md with findings

---

## Success Criteria

✅ All relevant files identified and documented
✅ Current model-viewer configuration understood
✅ Media gallery flow mapped with diagrams
✅ Modification points clearly identified
✅ Analysis report created with actionable insights
✅ No changes made to codebase (read-only analysis)

---

## Risk Assessment

**Low Risk Phase** - Read-only analysis, no code changes

**Potential Issues:**
- Large file sizes (main-product.liquid is ~27k tokens) → Use chunked reading or grep
- Complex Liquid logic → Document step-by-step for clarity
- Nested snippet includes → Create visual map

**Mitigation:**
- Read files in chunks if >2000 lines
- Use Grep for specific patterns (model-viewer, first_3d_model, DeferredMedia)
- Create clear diagrams in analysis report

---

## Security Considerations

None for this phase (read-only analysis)

---

## Next Steps

After analysis complete:
→ **Phase 02:** Prepare 3D assets (optimize GLB, create fallback images)
→ Document which files need modification
→ Create backup/version control checkpoint
