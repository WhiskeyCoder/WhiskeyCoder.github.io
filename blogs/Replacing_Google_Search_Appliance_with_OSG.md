# Replacing Google Search Appliance with Open Source: Meet OSG Search

## The Death of GSA and the Birth of Something Better

When Google pulled the plug on the **Google Search Appliance (GSA)** in 2019, they left a crater in the enterprise search landscape. For years, GSA had been the gold standard—a turnkey solution that gave organizations Google-quality search over their own data. It was fast, familiar, and it *just worked*.

Then it didn't.

Organizations scrambled. Some migrated to Google's cloud offerings with hefty price tags and vendor lock-in. Others cobbled together Elasticsearch or Solr deployments with janky front-ends that looked like they were designed in 2003. The problem wasn't just finding a replacement—it was finding one that didn't feel like a massive downgrade.

I wanted something different:

- **Clean, Google-like interface** that users would actually want to use
- **Self-hosted and open source** so I control the data and the stack
- **Flexible enough** to search files, feeds, metadata, images—anything
- **Built for 2025**, not 2005

That's why I built **OSG Search** (Open Source Google Search).

---

## What the Hell is OSG Search?

OSG Search is a **modern, open-source web search front-end** powered by **OpenSearch** and designed to work seamlessly with **Apache Tika** pipelines. 

Think of it as GSA's spiritual successor, but without the $500k hardware appliance or the proprietary bullshit.

It takes JSON documents from any indexing pipeline and renders them into a **familiar Google-style search experience**—complete with tabs for Net, Images, Maps, Shopping, Videos, News, and Books. If your documents are indexed in OpenSearch, OSG Search will make them look damn good.

### The Stack

- **Frontend**: React + TypeScript (clean, fast, no bloat)
- **Backend**: OpenSearch (fork of Elasticsearch, fully open source)
- **Ingestion**: Apache Tika (extracts text, metadata, images from any file format)
- **UI Philosophy**: Google SERP aesthetics with dark/light theme support

---

## Key Features That Actually Matter

### ✨ Google-Like UI That Doesn't Suck

OSG Search gives you the familiar SERP layout you've been conditioned to expect:

- **Net**: Primary search results with compact cards and 150-char snippets
- **Images**: Grid view with click-to-preview functionality
- **Maps**: Full-width Leaflet map with theme-aware tiles
- **Shopping**: Auto-detects items with `price` fields
- **Videos**: Supports YouTube, Vimeo, and local video files
- **News**: Placeholder for RSS integration (coming soon)
- **Books**: Filters `.pdf` and `.epub` documents automatically

The interface adapts based on what fields exist in your indexed documents. Got `image_url` or `image_base64`? Images tab. Got `price` and `currency`? Shopping tab. Got `lat` and `long`? Maps tab. It's smart enough to route content where it makes sense.

### 🔍 Multi-Tab Search Intelligence

Each tab isn't just a filter—it's a specialized view optimized for different content types:

**Net Tab**: Your primary SERP. Tight, minimal cards with titles, URLs, snippets, and timestamps. This is where most of your text-based content lives.

**Images Tab**: Renders `image_base64`, `image_url`, or `thumbnail_url` fields in a responsive grid. Click any image to open the full JSON document view with all metadata preserved.

**Maps Tab**: Uses Leaflet.js to plot documents with location data. Supports both explicit `lat/long` fields and geocoded `Address` + `Country` combinations. Theme-aware tiles (dark mode uses dark map tiles, light mode uses standard).

**Shopping Tab**: Automatically surfaces documents containing `price` and `currency` fields. Perfect for product catalogs, inventory databases, or price monitoring pipelines. Shows "Nothing for sale" when no results match.

**Videos Tab**: Basic detection for YouTube, Vimeo, and local video formats (`.mp4`, `.webm`, `.ogg`). Displays video thumbnails and metadata.

**Books Tab**: Filters documents by file extension (`.pdf`, `.epub`). Great for document repositories, research libraries, or academic archives.

### 📄 Structured JSON Reader

One of the slickest features: OSG Search includes a built-in document viewer that takes raw JSON and makes it readable.

When you click into a search result, the viewer:

- Formats nested JSON into clean, scannable sections
- Auto-paragraphs long text blocks for readability
- Preserves image previews and metadata
- Shows document source paths and timestamps

No more squinting at minified JSON blobs. The reader makes structured data feel like prose.

### ⚡ Direct OpenSearch Queries

OSG Search talks directly to your OpenSearch instance—no middleware, no abstraction layers, no bullshit.

Default connection: `http://localhost:9200`  
Default index: `file_index`

All queries are executed via the OpenSearch REST API using modern `fetch` calls. The frontend sends the query, OpenSearch returns JSON, and OSG Search renders it. Simple, transparent, hackable.

### 🔓 Open and Contribution-Friendly

Unlike most web apps that disable dev tools and lock down the UI, OSG Search is built for tinkerers:

- **Dev tools enabled** (right-click inspect works)
- **Text selection enabled** (copy/paste works)
- **No obfuscation** (readable source code)
- **MIT License** (do whatever you want)

Fork it. Customize it. Break it. Build something better. That's the point.

---

## How It Actually Works

### The Ingestion Pipeline

1. **Documents enter the system** via Apache Tika (or any other extraction tool)
2. **Tika extracts** text, metadata, images, and structured data
3. **JSON output** is indexed into OpenSearch
4. **OSG Search queries** the index and renders results

### The Document Schema

OSG Search is designed to work with flexible JSON schemas. Minimal shape:

```json
{
  "title": "Document Title",
  "url": "https://example.com/document",
  "content": "Full document content...",
  "snippet": "Short summary...",
  "created_date": "2024-01-01T00:00:00Z",
  "file_path": "/ingest/source.json"
}
```

Enriched shape (unlocks all tabs):

```json
{
  "title": "Product Name",
  "url": "https://example.com/product",
  "content": "Full description...",
  "snippet": "Short summary...",
  "raw_content": "{ \"title\": \"...\", \"summary\": \"...\" }",
  "image_url": "https://example.com/image.jpg",
  "image_base64": "iVBORw0KGgo...",
  "thumbnail_url": "https://example.com/thumb.jpg",
  "Address": "1600 Amphitheatre Pkwy",
  "Country": "United States",
  "Lat": 37.422,
  "Long": -122.084,
  "price": 19.99,
  "currency": "USD",
  "created_date": "2024-01-01T00:00:00Z",
  "file_path": "/ingest/source.json"
}
```

The more fields you index, the richer the search experience.

### The Query Flow

1. User types query into search bar
2. Frontend sends query to OpenSearch via REST API
3. OpenSearch executes full-text search across indexed fields
4. Results are returned as JSON
5. Frontend routes results to appropriate tabs based on field presence
6. User clicks result → structured reader view opens

Fast. Clean. No vendor APIs. No rate limits. No cloud costs.

---

## Quickstart: Get It Running in 5 Minutes

### Prerequisites

- **Node.js** (v18+)
- **OpenSearch** instance running (default: `http://localhost:9200`)
- Documents indexed in your target index (default: `file_index`)

### Installation

```bash
git clone https://github.com/WhiskeyCoder/osg-search.git
cd osg-search
npm install
npm start
```

The app will be live at `http://localhost:3000`.

### Optional: Custom Configuration

Create a `.env` file if you need non-default settings:

```env
REACT_APP_OPENSEARCH_URL=http://localhost:9200
REACT_APP_OPENSEARCH_INDEX=your_custom_index
```

### Building for Production

```bash
npm run build
```

This generates a production-ready static site in the `build/` folder. Serve it with nginx, Apache, or any static hosting service.

---

## Why Use OSG Search?

### You Miss GSA But Want Modern UX

GSA was great for 2008. OSG Search is built for 2025. Same familiar interface, but with dark mode, responsive design, and none of the proprietary hardware.

### You Want Self-Hosted Search You Control

No vendor lock-in. No cloud costs. No data leaving your infrastructure. Run it on your homelab, your VPS, or your enterprise network. Your data, your rules.

### You Deal with Files, JSON, or Feeds

If you're indexing documents, scraping data, or building intelligence pipelines, OSG Search turns your raw data into a searchable interface in minutes.

### You Want a Hackable Base for Search-Driven Projects

OSG Search is a foundation, not a black box. Build OSINT dashboards, internal wikis, e-commerce search, research portals—whatever you need. The code is clean, the architecture is simple, and the license is permissive.

---

## Roadmap: What's Coming Next

### Vector Search

Add embeddings-based semantic search powered by OpenSearch's vector capabilities. Find documents by meaning, not just keywords.

### Image Similarity Search

Upload an image, find similar images in your index. Perfect for OSINT investigations, product catalogs, or media libraries.

### RSS/News Integration

Pull live feeds into the News tab. Index them automatically and search across real-time content.

### Analytics & Click Feedback

Track query patterns, click-through rates, and result relevance. Use the data to improve ranking and refine your index.

### Advanced Filters

Faceted search by file type, date range, source domain, and custom metadata fields.

---

## Real-World Use Cases

### OSINT & Intelligence Analysis

Index scraped data, leaked documents, or public records. Search across massive datasets with a familiar interface. Filter by location, date, or content type.

### Self-Hosted Knowledge Base

Replace Confluence, Notion, or Google Drive search with something you control. Index internal docs, wikis, and files. Give your team a fast, private search experience.

### E-Commerce & Product Catalogs

Index product data with prices, images, and metadata. Customers search products directly from your OpenSearch instance—no third-party services required.

### Academic & Research Libraries

Index PDFs, EPUBs, and academic papers. Scholars search by title, author, keyword, or full-text content. Maps tab shows research by geographic relevance.

### Media Archives

Index photos, videos, and metadata. Journalists, historians, and archivists search by date, location, or visual similarity.

---

## Get Involved

OSG Search is open source under the **MIT License**. Fork it. Hack it. Contribute PRs. Report bugs. Request features.

👉 **GitHub**: [github.com/WhiskeyCoder/osg-search](https://github.com/WhiskeyCoder/osg-search)

👉 **Issues**: Found a bug or have an idea? [Open an issue](https://github.com/WhiskeyCoder/osg-search/issues)

👉 **Discussions**: Join the conversation and share your use cases

---

## Final Thoughts

Google killed GSA, but they didn't kill the need for fast, familiar, self-hosted search. OSG Search brings that experience back—without the vendor lock-in, cloud costs, or proprietary hardware.

It's built on open standards (OpenSearch, Apache Tika), uses modern tools (React, TypeScript), and respects your data (self-hosted, MIT licensed).

**Open source search doesn't have to feel clunky.** With OSG Search, you get the familiar Google experience—but powered by OpenSearch, Apache Tika, and your own data.

Try it. Break it. Make it yours.

---

*Built by [@WhiskeyCoder](https://github.com/WhiskeyCoder) | MIT License | Contributions welcome*
