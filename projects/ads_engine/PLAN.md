# Ads Engine - Implementation Plan & Progress

## Project Status: Phase 1 Complete ✓, Phase 2 In Progress

### Completed

- [x] **Setup**
  - [x] Update .gitignore for ads_engine project
  - [x] Create folder structure (local/, scripts/)
  - [x] Write download_dataset.py script
  - [x] Write process_ads.py script
  - [x] Create scripts/README.md

- [x] **Data Processing**
  - [x] Download AdImageNet dataset (9,003 ads)
  - [x] Generate ads.json with keywords and categories

- [x] **Frontend V1 (Personalization)**
  - [x] Build index.html with embedded CSS
  - [x] Updated design to match main site minimalism (warm ivory, IBM Plex Sans)
  - [x] Implement AdMatcher class with scoring algorithm
  - [x] Add user input form
  - [x] Display personalized ads based on keywords
  - [x] Fallback to random ads for poor matches

- [x] **Frontend V2 (Dimension-Based Layouts)**
  - [x] Analyzed dataset: 8 distinct ad sizes (300x250, 728x90, 300x600, 970x250, etc.)
  - [x] Defined 4 size buckets: rect, leaderboard, tall, billboard
  - [x] Pre-index ads by size bucket on load for fast selection
  - [x] Dimension-based layout system: all ads in a layout are the same size
  - [x] 4 CSS layouts optimized per bucket (2x2 grid, vertical stack, side-by-side, etc.)
  - [x] Hybrid pool selection: score ads per bucket, pick best bucket by avg score
  - [x] Layout variety: random selection among similarly-scoring buckets
  - [x] Fallback: random bucket on initial load (not always rect)
  - [x] Responsive design (collapse to single column on mobile)

- [x] **Debugging & Polish**
  - [x] Add toggleable debug mode (showDebugInfo flag in index.html)
  - [x] Discovered ad blocker issue (ERR_BLOCKED_BY_CLIENT), documented
  - [x] Removed category display from ad cards
  - [x] Fixed viewport sizing (max-width per layout, constrained image heights)

- [x] **Documentation**
  - [x] Write project README.md
  - [x] Write PLAN.md (this file)
  - [x] Document ad blocker issue in README
  - [x] Updated CLAUDE.md with ads_engine project

### Current Work

**Layout Testing & Refinement** (In Progress)
- Verify all 4 layouts display correctly (rect, leaderboard, tall, billboard)
- Test search-based layout selection with various queries
- Fine-tune spacing and sizing per layout

### Next Steps

1. **Final Polish**
   - Fine-tune layout sizing and spacing
   - Test responsive design thoroughly
   - Test personalization with various queries

2. **Deploy to GitHub Pages** (When Ready)
   - Commit `ads.json` (images stay local/gitignored)
   - Push to GitHub
   - Test at `https://dylanoconnell.com/projects/ads_engine/`
   - **Note**: Images will be broken on GitHub Pages until S3 setup

### Future Phases

#### Phase 2: AWS S3 Migration

When ready to make this publicly accessible:

1. Create S3 bucket (or Cloudflare R2)
2. Upload 9,003 images with organized naming
3. Update `ads.json` to use S3 URLs
4. Test production deployment
5. Add link from main site (index.html Miscellany section)

**Cost Estimate:**
- S3: ~$0.01/month storage + minimal egress
- Cloudflare R2: $0.015/GB storage, zero egress fees (recommended)

#### Phase 3: Enhanced Matching

- Use text embeddings (e.g., sentence-transformers)
- Semantic similarity instead of keyword matching
- Better handling of multi-word categories
- User feedback loop

#### Phase 4: Polish

- UI animations and transitions
- Dark mode toggle
- Share functionality
- Analytics tracking

## Technical Decisions

### Why Keyword-Based Matching?

- Simple and transparent
- No external dependencies
- Works entirely client-side
- Good enough for V1 humor/experiment

### Why Local Images + S3 Later?

- Git shouldn't store 450MB of images
- Testing locally doesn't need public hosting
- S3/R2 is cheap when ready to deploy
- Keeps repo lightweight

### Why Self-Contained HTML?

- Follows site philosophy (no frameworks)
- Easy to maintain
- Single file deployment
- Pattern established by Oscars game

### Why AdImageNet Dataset?

- MIT license (permissive)
- Real programmatic ads (authentic)
- Good variety of categories
- Pre-extracted OCR text
- 9,003 ads (large enough for variety)

## Performance Targets

- ads.json load: < 1 second
- Initial render: < 500ms
- Personalization: < 300ms
- Works on 3G mobile connections

## Dataset Size Distribution

| Dimensions | Count | Bucket | Layout |
|---|---|---|---|
| 300×250 | 3,355 | rect | 2×2 grid (4 ads) |
| 728×90 | 2,827 | leaderboard | Vertical stack (4 ads) |
| 300×600 | 941 | tall | Side by side (3 ads) |
| 970×250 | 857 | billboard | Vertical stack (3 ads) |
| 160×600 | 352 | tall | Side by side (3 ads) |
| 970×90 | 326 | leaderboard | Vertical stack (4 ads) |
| 336×280 | 184 | rect | 2×2 grid (4 ads) |
| 320×50 | 161 | leaderboard | Vertical stack (4 ads) |

## Testing Checklist

- [x] Dataset downloads successfully
- [x] ads.json generates correctly
- [x] Local server loads page
- [x] Random ads display (no input)
- [x] Personalized matching works
- [ ] All 4 layouts display correctly
- [ ] Responsive on mobile/tablet/desktop
- [ ] No console errors (with ad blocker disabled)
- [x] Images load properly (with ad blocker disabled)

## Known Limitations

1. **Ad Blockers**: Block images due to folder name and dimension-based filenames. Users must disable ad blocker.
2. **No Images on GitHub Pages**: Images are gitignored; deployed version needs S3/R2 for image hosting.
3. **Simple Matching**: Keyword-based matching misses semantic similarity (e.g., "car" won't match "automotive")
4. **English Only**: OCR text and keywords assume English
5. **No Caching**: Fetches ads.json on every load
6. **Client-Side Only**: All processing in browser

## Questions for User

- Deploy to GitHub Pages now (without images) or wait for S3 setup?
- Add link from main site Miscellany section?
- Any layout adjustments needed?
