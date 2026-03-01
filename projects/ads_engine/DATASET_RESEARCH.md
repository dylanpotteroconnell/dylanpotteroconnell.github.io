# Dataset Expansion Research

The current ads_engine uses 9,003 ads from AdImageNet — all modern programmatic banner ads (300x250, 728x90, etc.). They work well but they're visually samey. This doc brainstorms options for adding vintage, illustrated, and otherwise weird/interesting ads from other sources.

**Goal**: At least 1,000+ ads from non-AdImageNet sources. Ideally a mix of eras and styles.

---

## Source Comparison

| Source | Volume | Era | License | Download Method | OCR Included? | Effort |
|--------|--------|-----|---------|----------------|---------------|--------|
| **Newspaper Navigator** | 100K+ ad crops | 1850–1963 | Public domain | Yearly zip files | No (page-level OCR available separately) | Low |
| **Duke Ad\*Access** | 7,219 | 1911–1955 | Research use w/ attribution | IIIF manifests (scriptable) | No | Medium |
| **Duke EAA** | 9,000+ | 1850–1920 | Research use w/ attribution | IIIF manifests (scriptable) | No | Medium |
| **biglam/illustrated_ads** | 549 | 1850–1963 | CC0 | `load_dataset()` | Yes | Trivial |
| **Wikimedia Commons** | ~2,000+ | Mixed | CC / Public domain | Imker / CommonsDownloader | No | Low |
| **Smithsonian Open Access** | Large (subset is ads) | Mixed | CC0 | REST API | No | Medium |
| **Internet Archive** | Hundreds–thousands | Mixed | Mostly public domain | `ia` CLI tool | No | Low |
| **Flickr vintage ad groups** | 145K+ photos | Mixed | Mixed (filter by CC) | Flickr API | No | Medium |
| **Reddit r/vintageads** | Tens of thousands | Mixed | Unclear | BDFR tool | No | Low |
| **Pitt Ads Dataset** | 64,832 | Mixed/modern | Email-gated academic | Email authors for URLs | No | Medium |

---

## Tier 1: Best Starting Points

### Newspaper Navigator (Library of Congress)

The single largest source. The Library of Congress ran an object detection model over 16 million historic newspaper pages from Chronicling America and extracted cropped images for 7 content types, including **advertisements**.

- **What you get**: Pre-cropped rectangular ad images from American newspapers, 1850–1963. Black and white, newspaper scan quality. Authentically vintage.
- **Volume**: Hundreds of thousands of ad crops. The full dataset spans millions of detected regions across all categories.
- **License**: Public domain, unrestricted reuse.
- **How to download**: Yearly zip files from `https://news-navigator.labs.loc.gov/prepackaged/{year}_advertisements.zip`. Sample datasets (1,000 random images per type per year) also available.
- **Metadata**: JSON/CSV with bounding boxes, confidence scores, source page info.
- **OCR**: Not per-ad, but the underlying Chronicling America pages have full OCR in ALTO XML. You'd need to run your own OCR on the crops, or cross-reference with the page-level OCR.
- **Caveats**: The project was retired in 2025, but the data remains on GitHub/S3. Quality varies — some crops are clean, others include partial surrounding content. Confidence filtering (e.g., >0.9) helps.
- **Links**: [GitHub](https://github.com/LibraryOfCongress/newspaper-navigator), [data page](https://bcglee.github.io/newspaper-navigator.html), [paper](https://arxiv.org/abs/2005.01583)

**Quickstart idea**: Download a single year's ad zip (e.g., 1920) to see what the images look like. If they're good, scale up.

### Duke University: Ad\*Access

Over 7,000 ads from US and Canadian newspapers/magazines, 1911–1955. Curated by the Hartman Center for Sales, Advertising & Marketing History.

- **What you get**: Scanned print ads in 5 subject areas: Radio, Television, Transportation, Beauty & Hygiene, World War II.
- **Volume**: 7,219 ads.
- **License**: Available for research, teaching, and private study. Reproduce with attribution.
- **How to download**: Duke Digital Repository with IIIF manifests. Would need a script to iterate through manifests and download images.
- **Image quality**: Good. These are curated archival scans, not auto-detected crops.
- **OCR**: None included — would need to run your own.
- **Links**: [Collection](https://repository.duke.edu/dc/adaccess), [Research guide](https://guides.library.duke.edu/adaccess)

### Duke University: Emergence of Advertising in America (1850–1920)

Over 9,000 images from the dawn of American advertising. This is the "weird and wonderful" source.

- **What you get**: Trade cards, patent medicine ads, illustrated catalogs, calendars, almanacs, billboards, leaflets. Very different from modern banner ads.
- **Volume**: 9,000+ items.
- **License**: Same as Ad\*Access — research/teaching use with attribution.
- **How to download**: IIIF manifests via Duke Digital Repository.
- **Image quality**: High-resolution archival scans.
- **Links**: [Collection](https://repository.duke.edu/dc/eaa)

### biglam/illustrated_ads (HuggingFace)

A small sampler dataset — 549 cropped ad images from Newspaper Navigator, already packaged for easy Python access.

- **What you get**: 549 illustrated newspaper ads with OCR text, bounding boxes, confidence scores, and source metadata.
- **License**: CC0.
- **How to download**: `pip install datasets` then `load_dataset("biglam/illustrated_ads")`. Takes under a minute.
- **Why it's useful**: Perfect for prototyping the processing pipeline before scaling up to larger sources. Also a good sanity check for what Newspaper Navigator ads actually look like.
- **Links**: [HuggingFace](https://huggingface.co/datasets/biglam/illustrated_ads)

---

## Tier 2: Good Supplementary Sources

### Wikimedia Commons

Advertisement categories contain a few thousand freely-licensed images across all eras.

- **Key categories**: [Advertisements](https://commons.wikimedia.org/wiki/Category:Advertisements) (29 subcategories), [Advertising posters](https://commons.wikimedia.org/wiki/Category:Advertising_posters), decade-specific categories like [1890s advertisements](https://commons.wikimedia.org/wiki/Category:1890s_advertisements).
- **License**: All CC or public domain — cleanest licensing of any source.
- **Download tools**: [Imker](https://commons.wikimedia.org/wiki/Commons:Imker_(batch_download)) (Java), [CommonsDownloader](https://commons.wikimedia.org/wiki/Commons:Download_tools) (Python), or Wikimedia API directly.
- **Volume**: Probably 2,000–5,000 total across subcategories. Not huge, but everything is clean.

### Smithsonian Open Access

2.8 million CC0 images. The trade literature collection alone has 500K+ items including advertising brochures and catalogs from 30,000+ companies.

- **License**: CC0 — completely unrestricted.
- **How to access**: REST API (free key from api.data.gov). Bulk JSON metadata on [GitHub](https://github.com/Smithsonian/OpenAccess).
- **Challenge**: Not curated specifically for ads. You'd need to search/filter metadata for advertising-related items. Could be very rewarding if you find the right queries.

### Internet Archive

Various user-contributed collections of scanned vintage ads.

- **Collections**: [Vintage American Print Ads](https://archive.org/details/vintage-american-print-ads), [50 Vintage Newspaper Print Ads (60s & 70s)](https://archive.org/details/VintageNewspaperPrintAdvertisements), [Vintage Computer Advertising](https://archive.org/details/VintageComputerAdvertisingD.D.TeoliJr.A.C.1).
- **Download tool**: `pip install internetarchive`, then use `ia download` or `ia search`.
- **Volume**: Hundreds to low thousands across collections. Variable quality.
- **License**: Mostly public domain or CC. Check per-item.

### Library of Congress (Direct)

Beyond Newspaper Navigator, the LOC has direct collections:

- **Advertising Poster Collection**: Posters for books, fairs, railroads, patent medicines, 1845–1947.
- **WPA Posters**: 907 stunning graphic design posters, 1936–1943. Not strictly commercial ads, but visually striking.
- **API**: Free, no key required. [loc.gov/apis](https://www.loc.gov/apis/)

---

## Tier 3: Large But Messy

### Flickr Vintage Ad Groups

The [Vintage Advertising group](https://www.flickr.com/groups/vintage_advertising/) has 145K+ photos and 8,400 members. Other groups: Golden Age of Advertising (1950s–1970s), Ads Through the Ages, Smooth Smoke Slogans.

- **Download**: Flickr API with `flickr.groups.pools.getPhotos`. Filter by CC license using the `license` parameter.
- **Problem**: Most photos are "All Rights Reserved." After CC filtering, volume drops significantly. Image quality is inconsistent (phone photos of magazine pages mixed with proper scans).

### Reddit r/vintageads

218K+ members, years of posts. Use [BDFR](https://github.com/Serene-Arc/bulk-downloader-for-reddit) to bulk download:

```bash
pip install bdfr
python -m bdfr download ./vintageads --subreddit vintageads --sort top --time all
```

- **Volume**: Tens of thousands of images.
- **Problem**: Licensing is a mess. Most are scans of copyrighted magazine ads uploaded without permission. Fine for a personal project, problematic for anything public-facing.

### Pitt Ads Dataset (Academic)

64,832 ad images with rich annotations (topic, sentiment, slogans, Q&A). Mostly modern magazine/commercial ads, not exclusively vintage.

- **Access**: Email kovashka@cs.pitt.edu for image URLs. Annotations downloadable as zip.
- **Good for**: If you want a large, well-annotated dataset and don't mind the email-gating.

---

## Processing Challenges

### 1. Image Sizing

AdImageNet comes in standard IAB ad sizes (300x250, 728x90, etc.). Vintage ads will be arbitrary sizes — full-page magazine scans, cropped newspaper regions, trade cards, posters.

**Solution**: Batch processing with Pillow.

```python
from PIL import Image, ImageOps

# Option A: Crop to fill a target size (loses edges)
fitted = ImageOps.fit(img, (300, 250), method=Image.LANCZOS)

# Option B: Pad to fit (letterbox with background color)
padded = ImageOps.pad(img, (300, 250), color=(255, 255, 255), method=Image.LANCZOS)

# Option C: Just classify by aspect ratio, don't force to standard sizes
ratio = width / height
if ratio > 2.5: category = "wide"
elif ratio < 0.5: category = "tall"
else: category = "square"
```

**Recommendation**: Option C is probably best. The current layout system already handles wide/tall/square aspect ratios. No need to force vintage ads into IAB sizes — just classify them and let the layout templates handle it. Pad minimally to avoid distortion.

### 2. OCR / Text Extraction

AdImageNet has OCR text from Google Vision API. Most vintage sources don't include OCR.

**Options (ranked by practicality)**:

1. **PaddleOCR** (free, open source) — `pip install paddlepaddle paddleocr`. Good accuracy out of the box, handles diverse fonts reasonably well.
2. **Multimodal LLM** (Claude or GPT-4 Vision) — send the image and ask for text extraction. Best for stylized/decorative vintage fonts. Costs a few cents per image.
3. **Tesseract** (`pip install pytesseract`) — free but struggles with decorative fonts. Needs preprocessing (deskew, binarize, 300 DPI input).
4. **Google Vision API** — best accuracy, 1,000 images/month free tier.

**Recommendation**: Use PaddleOCR as the default. For images where PaddleOCR returns garbage (decorative vintage fonts), fall back to a multimodal LLM. For a batch of 1,000–2,000 images this is very manageable.

### 3. Extracting Ads from Full Pages

Some sources (Chronicling America raw pages, magazine scans) give you full pages, not individual ads. You'd need to detect and crop ad regions.

**Best tool**: [LayoutParser](https://layout-parser.github.io/) with the Newspaper Navigator model. Pre-trained to detect ads (among other content types) on historical newspaper pages.

```python
import layoutparser as lp
model = lp.Detectron2LayoutModel(
    'lp://NewspaperNavigator/faster_rcnn_R_50_FPN_3x/config',
    label_map={0:"Photograph", 1:"Illustration", 2:"Map", 3:"Comic",
               4:"Editorial Cartoon", 5:"Headline", 6:"Advertisement"}
)
layout = model.detect(image)
ad_blocks = [b for b in layout if b.type == "Advertisement"]
```

**Caveat**: Requires Detectron2 + PyTorch. Works best on Linux/WSL. The model was trained on newspapers, not magazines — accuracy on magazine layouts will vary.

**Recommendation**: Skip this for now. Newspaper Navigator already provides pre-cropped ads. Only needed if you want to process raw scans from other sources.

### 4. Category Assignment

The current `process_ads.py` assigns categories via keyword matching against OCR text. This same pipeline works for new ads — just run OCR first, then the existing categorizer.

For vintage ads, you might want to add categories like "Patent Medicine", "Tobacco", "War Bonds" — things that don't map well to the current 12 modern categories (Technology, Finance, Travel, etc.). But that's a future concern.

### 5. Metadata Format

New ads need to match the existing `ads.json` schema:

```json
{
  "id": "vintage_00001",
  "image_path": "local/images/vintage/...",
  "text": "OCR extracted text...",
  "dimensions": [width, height],
  "keywords": ["extracted", "keywords"],
  "categories": ["Category Name"],
  "source": "newspaper_navigator"  // NEW: track provenance
}
```

Adding a `source` field would help distinguish vintage ads from AdImageNet ads in the UI and for debugging.

---

## Copyright Notes

| Era | Status |
|-----|--------|
| Pre-1930 | Public domain (safe) |
| 1930–1977, no (c) notice | Likely public domain |
| 1930–1977 with (c), no renewal | Likely public domain (many ads weren't renewed) |
| 1930–1977 with (c), renewed | Copyrighted for 95 years |
| Post-1978 | Copyrighted (automatic) |

For a personal project, the safest path is to stick with pre-1930 material and explicitly-licensed sources (CC0, public domain). Practically, most vintage ads from before the 1970s are in a gray zone where enforcement is extremely unlikely.

---

## Recommended Next Steps

### Phase 1: Quick prototype (1–2 hours)

1. Download `biglam/illustrated_ads` from HuggingFace (549 images, trivial).
2. Write a processing script to convert them to the `ads.json` format.
3. Test them in the existing UI alongside the AdImageNet ads.
4. See how the layout handles non-standard sizes.

### Phase 2: Scale up with Newspaper Navigator (half day)

1. Download 2–3 years of ad zips from Newspaper Navigator (e.g., 1920, 1940, 1950).
2. Filter by confidence score (>0.9) to get clean crops.
3. Run PaddleOCR on the crops.
4. Process into `ads.json` format and merge with existing data.
5. This alone could add thousands of vintage newspaper ads.

### Phase 3: Duke collections (half day)

1. Write a script to download images via IIIF manifests from Duke Ad\*Access and/or EAA.
2. These are higher-quality, more visually interesting ads (color magazine ads, illustrated trade cards).
3. Process and merge.

### Phase 4: Diversify further (ongoing)

- Add Wikimedia Commons ads for variety.
- Try Smithsonian Open Access API for trade cards and ephemera.
- Browse Internet Archive collections for niche finds (vintage computer ads, etc.).

---

## Open Questions

- **How many total ads do we want?** 9K from AdImageNet + a few thousand vintage? Or much larger?
- **Should vintage ads be visually distinguished in the UI?** (e.g., a "vintage" badge, sepia tint, different section)
- **Storage**: Vintage images go in `local/images/` alongside AdImageNet? Or a separate subfolder like `local/images/vintage/`?
- **ads.json size**: Adding thousands more ads increases the JSON payload. At what point do we need to split it or add lazy loading?
- **Categories**: Do we need vintage-specific categories, or force-fit into the existing 12?
