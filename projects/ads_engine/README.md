# Ads Engine

A humorous experiment in personalized advertising. Enter your interests and see real programmatic ads matched to your profile using keyword-based categorization.

## Features

- **9,003 Real Ads**: Uses the AdImageNet dataset (MIT license) containing actual programmatic display ads
- **Personalized Matching**: Keyword-based algorithm scores ads based on your interests
- **Dimension-Aware Layouts**: 4 layout templates optimized for different ad sizes (rect, leaderboard, tall, billboard)
- **12 Product Categories**: Technology, Finance, Travel, Food & Dining, Fashion & Beauty, Health & Fitness, Entertainment, Home & Garden, Automotive, Education, Shopping & Retail, Services
- **Responsive Design**: Layouts adapt to mobile/tablet/desktop
- **Zero Dependencies**: Plain HTML, CSS, and JavaScript

## How It Works

1. **User Input**: Type interests like "technology travel fitness"
2. **Keyword Extraction**: Extract meaningful keywords from input
3. **Ad Scoring**: Score each ad based on:
   - Direct keyword overlap (40%)
   - Category match (40%)
   - Text similarity (20%)
4. **Layout Selection**: Ads are grouped by size bucket (rect, leaderboard, tall, billboard). The bucket with the best-scoring ads is selected, ensuring all displayed ads share the same dimensions for a clean layout.
5. **Display**: Show 3-4 ads in an optimized grid for that size

## Project Structure

```
ads_engine/
├── index.html           # Self-contained frontend (HTML + CSS + JS)
├── ads.json            # Ad metadata with keywords/categories (~500KB)
├── local/              # Gitignored
│   ├── images/        # 9K ad images for local testing
│   └── ads_metadata.json
├── scripts/            # Gitignored Python processing scripts
│   ├── download_dataset.py
│   ├── process_ads.py
│   └── README.md
├── PLAN.md            # Implementation tracking
└── README.md          # This file
```

## Setup for Development

### 1. Install Python Dependencies

```bash
pip install datasets pillow
```

### 2. Download Dataset

```bash
cd scripts
python download_dataset.py
```

This downloads 9,003 images (~450MB) to `local/images/`.

### 3. Process Ads

```bash
python process_ads.py
```

This generates `ads.json` with keywords and categories.

### 4. Run Local Server

The project uses `fetch()` to load `ads.json`, which requires HTTP (not file://).

```bash
cd ../..  # Back to personal_site root
python -m http.server 8000
```

Then visit: `http://localhost:8000/projects/ads_engine/`

## Dataset

- **Source**: [AdImageNet on HuggingFace](https://huggingface.co/datasets/PeterBrendan/AdImageNet)
- **License**: MIT
- **Images**: 9,003 programmatic display ads
- **OCR Text**: Extracted via Google Vision API
- **Sizes**: Various (300x250, 728x90, 160x600, etc.)
- **Note**: Requires HuggingFace authentication to access

## Known Issues

- **Ad Blockers**: Browser ad blockers detect and block the images due to:
  - URL path containing "ads_engine"
  - Image filenames starting with ad dimensions (300x250, 728x90, etc.)
  - Users need to disable ad blockers for the site, or we need to rename folders/files for production
  - This is ironically fitting for a humorous ad display project!

## Future Improvements

- **Ad Blocker Workaround**: Rename `ads_engine` folder and image files to avoid ad blocker detection
- **AWS S3 Hosting**: Move images to S3 for production deployment
- **Better Matching**: Use embeddings or ML for semantic similarity
- **User Feedback**: Allow users to rate ad relevance
- **Analytics**: Track which categories get most interest
- **More Filters**: Filter by ad size, format, industry

## Tech Stack

- Pure HTML5, CSS3, JavaScript (ES6+)
- CSS Grid for responsive layout
- Fetch API for data loading
- No frameworks, no build tools

## Credits

Built by Dylan O'Connell as a humorous exploration of ad tech and personalization. Uses the AdImageNet dataset by PeterBrendan.
