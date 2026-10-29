# Pakistani Clothing Brands — Big Data Analytics

**YouTube comment collection, product metadata enrichment, and a foundation for sentiment analysis and price-aware clustering.**

> **Project type:** Week-wise Big Data Analytics coursework — **not** a Final Year Project (FYP).  
> **Status:** YouTube ingestion notebook developed; dataset collection and downstream model evaluation depend on execution and available data.  
> **Data goal:** Scalable collection with a *target* of 1M+ comment records, **not a claim that 1M records have been collected**.

## Project overview

This project investigates consumer reactions to Pakistani clothing brands by bringing together YouTube comments, video engagement metadata, and candidate product details. The longer-term objective is to enrich observations with **verified product prices**, classify English/Roman Urdu/Urdu sentiment, create product-level analytical features, and use clustering to identify patterns relevant to pricing and marketing.

The initial implementation focuses on **YouTube Data API v3** and produces **one flat CSV file** with one row per comment. It is designed to save progress incrementally and avoid loading the entire dataset into memory.

## Work completed so far

1. Defined the Big Data Analytics pipeline: collection → storage → data cleaning → product matching → sentiment analysis → product-level clustering → visualization.
2. Selected 10 initial Pakistani clothing-brand search terms: Khaadi, Sapphire, Gul Ahmed, Alkaram Studio, Nishat Linen, Limelight, Beechtree, Bonanza Satrangi, J., and Ethnic.
3. Integrated the **YouTube Data API v3** for paginated video discovery, video metadata, and top-level comment collection.
4. Added video-ID batching (up to 50 IDs per `videos.list` request) and comment pagination (up to 100 top-level comments per page).
5. Added keyword-based **product-category candidates** and extraction of **possible PKR price mentions** from video titles and descriptions.
6. Added SQLite storage, incremental commits, unique comment IDs, and restart/resume behavior to reduce duplicate collection.
7. Added a **single CSV export** containing comment text, video metadata, engagement, brand search term, product candidates, and placeholders for later ML features.
8. Added basic CSV inspection and chunk-based record counting for larger outputs.

**Not completed yet:** verified product-to-video matching, verified product prices, manual sentiment annotation, trained sentiment models, product-level clustering, and a deployed dashboard.

## Repository contents

```text
.
├── README.md
└── YouTube_Clothing_Million_Scale_Single_CSV.ipynb
```

The notebook generates these files during execution (do **not** commit large raw data or secrets):

```text
MyDrive/clothing_big_data/
├── youtube_collection.sqlite3
└── youtube_clothing_ONE_FILE.csv
```

## Technology stack

- **Python** — data collection and preprocessing
- **Google Colab / Jupyter** — notebook execution
- **YouTube Data API v3** (`google-api-python-client`) — videos and public comments
- **SQLite** — persistent intermediate storage and deduplication
- **Pandas** — inspection and chunked dataset checks
- **CSV** — one flat export file for the next analytical stages
- **Planned:** PySpark, TF-IDF + Logistic Regression, multilingual sentiment models, K-Means, and a dashboard

## Getting started (Google Colab)

1. Create a Google Cloud project, enable **YouTube Data API v3**, and create an API key: https://console.cloud.google.com/apis/credentials
2. Upload `YouTube_Clothing_Million_Scale_Single_CSV.ipynb` to https://colab.research.google.com/.
3. Run the notebook cells in order. When prompted, **enter only your API key** (not Python code).
4. Adjust the collection settings in the configuration cell:

   ```python
   BRANDS = ['Khaadi', 'Sapphire', 'Gul Ahmed', 'Alkaram Studio',
             'Nishat Linen', 'Limelight', 'Beechtree',
             'Bonanza Satrangi', 'J.', 'Ethnic']
   VIDEOS_PER_BRAND = 100
   MAX_COMMENTS_PER_VIDEO = 3000
   ```

5. If Google Drive is enabled in the notebook, authorize the mount and check `MyDrive/clothing_big_data/` for the output.
6. Re-run the collection cells as needed, respecting YouTube API quotas and terms. The notebook skips videos marked completed and deduplicates comments in SQLite.
7. Export `youtube_clothing_ONE_FILE.csv` and inspect it with chunked reads rather than opening the full file in spreadsheet software.

**Important:** YouTube searches also return influencer, review, and reseller channels. A search term is **not proof** that a video is from a brand's official account.

## Single CSV schema

Each row represents **one top-level YouTube comment**, combined with the associated video's metadata.

| Field(s) | Meaning |
| --- | --- |
| `platform`, `brand_search_term` | Source platform and brand used in the search; not a verified product label |
| `video_id`, `video_url`, `video_title`, `channel_id`, `channel_title` | Source video and channel information |
| `video_published_at`, `video_views`, `video_likes`, `reported_comment_count` | Published date and available engagement metadata |
| `comment_id`, `comment_text`, `comment_likes`, `comment_published_at`, `comment_updated_at` | Collected public top-level comment information |
| `product_candidate`, `product_category_candidate`, `product_url_candidate` | Product-identification hints requiring validation |
| `price_mention_pkr`, `price_mentions_count`, `price_source` | Possible PKR mentions in text, **not verified selling prices** |
| `verified_price_pkr`, `product_match_status` | Fields reserved for later catalog matching and verification |
| `sentiment`, `price_complaint` | Empty fields to populate after annotation/classification |
| `collected_at` | Export timestamp |

### Price and product verification

The YouTube API does **not** provide a standard clothing-product selling-price field. Extracted amounts can refer to discounts, multiple products, or unrelated mentions. Before price-based clustering, obtain product details from a permitted brand catalog or authorized data source and match each product using a verified product URL/SKU or another defensible identifier. Do not assign a single product to a multi-product video without supporting evidence.

## Big Data scale and limitations

- The pipeline is **designed to accommodate large exports**; it does not manufacture records or guarantee a million real comments.
- Collection volume depends on relevant video availability, whether comments are enabled, API access, daily quotas, request costs, and processing/storage resources.
- For reference, collecting **1,000,000 top-level comments** at 100 per page requires at least **10,000 comment-page requests**, in addition to video discovery and metadata requests.
- The notebook runs in Colab and uses SQLite. It is a practical prototype for sizeable data, **not a distributed multi-node ingestion system**. A future stage can migrate processing to Spark/Parquet or a managed data platform.
- A comment row is not a unique customer, sale, product, or verified purchaser.
- YouTube comments do not reliably reveal commenters' age, location, or purchasing history.

## Planned analysis

1. **Quality:** remove empty/spam comments, normalize text, examine language mix, and review duplicates.
2. **Sentiment:** manually label a representative sample (Positive / Negative / Neutral), evaluate a TF-IDF + Logistic Regression baseline, and consider a multilingual model for English, Roman Urdu, and Urdu.
3. **Price aspects:** separately label comments about price; price questions are not automatically negative.
4. **Product enrichment:** link only validated product records and verified prices to relevant observations.
5. **Clustering:** aggregate comments into **product-level features** (e.g., verified price, comment volume, positive/negative rates, price-complaint rate). Standardize numerical features, consider category encoding, and assess K-Means or alternative methods.
6. **Visualization:** compare product segments, sentiment distributions, brands, categories, and pricing patterns.

## Responsible data use

Use the official YouTube API within its terms and quotas. Do not publish API keys, access tokens, or other credentials. Respect platform retention/deletion requirements and minimize unnecessary personal information. Before uploading or redistributing large raw comment datasets, check whether doing so is permitted by the applicable API terms; publishing the collection code and schema may be safer than committing raw data.

## References

- YouTube Data API documentation: https://developers.google.com/youtube/v3
- YouTube quota costs: https://developers.google.com/youtube/v3/determine_quota_cost
- `search.list`: https://developers.google.com/youtube/v3/docs/search/list
- `videos.list`: https://developers.google.com/youtube/v3/docs/videos/list
- `commentThreads.list`: https://developers.google.com/youtube/v3/docs/commentThreads/list

## GitHub repository

**Repository name:** `pakistan-fashion-big-data-analytics`  
**Short description:** `A scalable YouTube API data collection pipeline for Pakistani clothing brands, combining public comments, video engagement, and product-price candidates for sentiment analysis and clustering.`

**GitHub link:** `[ADD YOUR GITHUB REPOSITORY URL HERE]`
