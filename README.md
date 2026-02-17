# AI Lead Generation System for n8n

A production-ready, modular n8n workflow system that automatically discovers and qualifies small local businesses as prospects for AI automation services.

> 💡 **AI-Assisted Development**: This workflow system is designed to work with the [n8n-MCP](https://github.com/czlonkowski/n8n-mcp) (Model Context Protocol) server, which gives AI assistants like Claude comprehensive access to n8n node documentation, properties, and operations. See the [n8n-MCP Integration](#-n8n-mcp-integration) section below for setup instructions.

## 🎯 Business Goal

This system creates an automated lead-generation pipeline that:

- **Finds small local businesses** that are strong candidates for AI automation services (chatbots, appointment booking, CRM automation, lead capture)
- **Focuses on low-competition niches** rather than saturated markets
- **Prioritizes underserved or digitally weak businesses** (outdated websites, no chatbot, poor SEO)
- **Uses AI to score and qualify leads** based on likelihood to purchase

### Target Niches

- Niche medical clinics (physiotherapy, podiatry, chiropractic)
- Specialty home services (garage door repair, pest control, pool cleaning)
- Local professional services (immigration consultants, tutoring centers, tax preparers)
- Healthcare services (dental, veterinary)
- Maintenance services (HVAC, plumbing, roofing, landscaping)

---

## 📊 Output Data

For each discovered business, the system collects:

| Field | Description |
|-------|-------------|
| Business Name | Company/practice name |
| Owner First Name | Inferred from website/email patterns |
| Email | Direct email (prioritizes personal over generic info@) |
| Phone | Business phone number |
| Website URL | Company website |
| City / Country | Business location |
| Niche Category | Industry classification |
| Lead Score | AI-generated 1-10 score |
| Prospect Reason | Why they're a good AI automation candidate |
| Suggested Services | Recommended AI services for outreach |

---

## 🏗️ Architecture Overview

The system uses a modular architecture with 5 interconnected workflows:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        MAIN WORKFLOW                                          │
│  ┌─────────────┐    ┌────────────────┐    ┌──────────────┐                   │
│  │   Manual    │───▶│    Setup       │───▶│    Split     │                   │
│  │   Trigger   │    │ Configuration  │    │   Queries    │                   │
│  └─────────────┘    └────────────────┘    └──────────────┘                   │
│         │                                        │                            │
│  ┌─────────────┐                                 ▼                            │
│  │  Schedule   │                         ┌──────────────┐                    │
│  │  Trigger    │───────────────────────▶│    Batch     │◀──────┐            │
│  │ (Weekly)    │                         │   Queries    │       │            │
│  └─────────────┘                         └──────┬───────┘       │            │
│                                                  │               │            │
│                                                  ▼               │            │
│                                          ┌──────────────┐       │            │
│                                          │   Search     │       │            │
│                                          │  Workflow    │       │            │
│                                          └──────┬───────┘       │            │
│                                                  │               │            │
│                                                  ▼               │            │
│                                          ┌──────────────┐       │            │
│                                          │ Rate Limit   │       │            │
│                                          │    Delay     │       │            │
│                                          └──────┬───────┘       │            │
│                                                  │               │            │
│                                                  ▼               │            │
│                                          ┌──────────────┐       │            │
│                                          │ Enrichment   │       │            │
│                                          │  Workflow    │       │            │
│                                          └──────┬───────┘       │            │
│                                                  │               │            │
│                                                  ▼               │            │
│                                          ┌──────────────┐       │            │
│                                          │  AI Scoring  │───────┘            │
│                                          │  Workflow    │                    │
│                                          └──────────────┘                    │
│                                                                               │
│  After all batches:                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌────────────┐ │
│  │ Deduplicate  │───▶│   Filter     │───▶│   Export     │───▶│   Notify   │ │
│  │   & Sort     │    │  Score ≥ 5   │    │  Workflow    │    │   Slack    │ │
│  └──────────────┘    └──────────────┘    └──────────────┘    └────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Sub-Workflows

#### 1. Search Workflow (`search-workflow.json`)
- Queries multiple data sources in parallel:
  - Google Places API
  - Yelp Fusion API  
  - SerpAPI (Google Maps)
- Normalizes results from different sources
- Filters invalid/incomplete records

#### 2. Enrichment Workflow (`enrichment-workflow.json`)
- Scrapes business websites for contact data
- Extracts emails from mailto: links and page content
- Identifies phone numbers from tel: links
- Infers owner names from page content
- Uses Hunter.io for additional email discovery
- Extracts social media profiles

#### 3. Scoring Workflow (`scoring-workflow.json`)
- AI-powered lead scoring using OpenAI GPT-4o-mini
- Alternative Claude integration available
- Rule-based fallback when AI is unavailable
- Scores leads 1-10 based on:
  - Lack of existing automation
  - Website quality indicators
  - Online presence strength
  - Business type suitability
  - Contact accessibility

#### 4. Export Workflow (`export-workflow.json`)
- Generates formatted Excel (.xlsx) files
- Sorts leads by score (highest first)
- Multiple delivery options:
  - Save to local disk
  - Upload to Google Drive
  - Email as attachment
- Generates summary statistics

---

## 📋 Node-by-Node Layout

### Main Workflow Nodes

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Manual Trigger | `manualTrigger` | Start workflow manually |
| 2 | Schedule Trigger | `scheduleTrigger` | CRON: Every Monday 9 AM |
| 3 | Setup Configuration | `code` | Define niches, locations, settings |
| 4 | Has Search Queries? | `if` | Validate configuration |
| 5 | Split Queries | `splitOut` | Separate into individual queries |
| 6 | Batch Queries | `splitInBatches` | Process one at a time |
| 7 | Execute Search | `executeWorkflow` | Call search sub-workflow |
| 8 | Rate Limit Delay | `wait` | 2-second delay between calls |
| 9 | Enrich Lead Data | `executeWorkflow` | Call enrichment sub-workflow |
| 10 | AI Lead Scoring | `executeWorkflow` | Call scoring sub-workflow |
| 11 | Deduplicate & Sort | `code` | Remove duplicates, sort by score |
| 12 | Filter by Score | `filter` | Only keep leads with score ≥ 5 |
| 13 | Export to Excel | `executeWorkflow` | Call export sub-workflow |
| 14 | Notify on Slack | `slack` | Send completion notification |

### Search Workflow Nodes

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Workflow Input | `executeWorkflowTrigger` | Receive query parameters |
| 2 | Prepare Search Query | `code` | Build API request parameters |
| 3 | Google Places API | `httpRequest` | Query Google Places |
| 4 | Yelp API Search | `httpRequest` | Query Yelp Fusion |
| 5 | SerpAPI Google Maps | `httpRequest` | Query SerpAPI |
| 6 | Normalize Results | `code` | Standardize data format |
| 7 | Filter Valid Results | `filter` | Remove empty records |
| 8 | Add Search Metadata | `code` | Tag with query info |

### Enrichment Workflow Nodes

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Workflow Input | `executeWorkflowTrigger` | Receive lead data |
| 2 | Prepare for Enrichment | `code` | Extract domain, set flags |
| 3 | Has Website? | `if` | Route based on website presence |
| 4 | Fetch Website | `httpRequest` | Download webpage HTML |
| 5 | Extract HTML Data | `html` | Parse with Cheerio selectors |
| 6 | Parse Website Data | `code` | Extract emails, phones, names |
| 7 | No Website Available | `code` | Handle missing websites |
| 8 | Merge Results | `merge` | Combine both paths |
| 9 | Hunter.io Lookup | `httpRequest` | Additional email discovery |
| 10 | Merge Hunter Data | `code` | Integrate Hunter results |
| 11 | Final Cleanup | `code` | Validate and format output |

### Scoring Workflow Nodes

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Workflow Input | `executeWorkflowTrigger` | Receive enriched lead |
| 2 | Prepare AI Analysis | `code` | Build analysis prompt |
| 3 | OpenAI Lead Analysis | `lmChatOpenAi` | AI scoring with GPT-4o-mini |
| 4 | Claude Lead Analysis | `lmChatAnthropic` | Alternative AI provider |
| 5 | Rule-Based Fallback | `code` | Backup scoring logic |
| 6 | Parse AI Response | `code` | Extract JSON from response |
| 7 | Merge Scoring Results | `merge` | Combine AI and fallback |
| 8 | Prepare Final Output | `code` | Format scored lead |

### Export Workflow Nodes

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Workflow Input | `executeWorkflowTrigger` | Receive all leads |
| 2 | Format for Excel | `code` | Prepare columns and data |
| 3 | Convert to Excel | `spreadsheetFile` | Generate .xlsx file |
| 4 | Save to Disk | `readWriteFile` | Write to filesystem |
| 5 | Upload to Google Drive | `googleDrive` | Cloud backup |
| 6 | Email Report | `emailSend` | Send as attachment |
| 7 | Generate Summary | `code` | Create export statistics |

---

## 🔌 Required Credentials & APIs

### Required (Choose at least one per category)

**Search APIs (at least one):**
- Google Places API - [Get API Key](https://console.cloud.google.com/)
- Yelp Fusion API - [Get API Key](https://www.yelp.com/developers/)
- SerpAPI - [Get API Key](https://serpapi.com/)

**AI Scoring (at least one):**
- OpenAI API - [Get API Key](https://platform.openai.com/)
- Anthropic Claude API - [Get API Key](https://console.anthropic.com/)

### Optional Enhancements

- Hunter.io - Email enrichment - [Get API Key](https://hunter.io/)
- Google Drive OAuth - Cloud storage
- Slack - Notifications
- SMTP - Email delivery

---

## 🚀 Setup Instructions for Beginners

### Step 1: Install n8n

```bash
# Using npm (Node.js required)
npm install n8n -g
n8n start

# Or using Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Step 2: Import Workflows

1. Open n8n at `http://localhost:5678`
2. Go to **Workflows** → **Import from File**
3. Import in this order:
   - `workflows/search-workflow.json`
   - `workflows/enrichment-workflow.json`
   - `workflows/scoring-workflow.json`
   - `workflows/export-workflow.json`
   - `workflows/ai-lead-generation-workflow.json` (main)

### Step 3: Configure Credentials

1. Go to **Settings** → **Credentials**
2. Add credentials for your chosen services:

**Google Places API:**
```
Name: Google Places API
Type: HTTP Query Auth
Parameter Name: key
Value: YOUR_API_KEY
```

**OpenAI:**
```
Name: OpenAI API
Type: OpenAI API
API Key: YOUR_OPENAI_KEY
```

**Yelp:**
```
Name: Yelp API
Type: HTTP Header Auth
Header Name: Authorization
Value: Bearer YOUR_YELP_API_KEY
```

### Step 4: Configure the Workflow

1. Open the main workflow `AI Lead Generation System`
2. Edit the **Setup Configuration** node
3. Customize:
   - `targetNiches`: Industries to search
   - `targetLocations`: Cities/regions to search
   - `maxResultsPerSearch`: Results per query
   - `rateLimitDelayMs`: Delay between API calls
   - `minLeadScore`: Minimum score threshold

### Step 5: Test the Workflow

1. Click **Execute Workflow** with the manual trigger
2. Monitor the execution in real-time
3. Check the output Excel file
4. Adjust configuration as needed

### Step 6: Activate Scheduled Runs

1. Toggle the workflow to **Active**
2. The schedule trigger runs every Monday at 9 AM
3. Customize the CRON expression as needed

---

## ⚖️ Ethical & Legal Compliance

This workflow is designed with ethics and compliance in mind:

### ✅ Data Sources
- Uses **publicly available business data** only
- Sources from legitimate business directories
- No scraping of personal/private information

### ✅ Rate Limiting
- Built-in 2-second delays between API calls
- Batch processing to prevent overwhelming servers
- Configurable rate limits

### ✅ Respectful Scraping
- Only fetches public business pages
- Respects robots.txt (when using HTTP requests)
- No login/authentication bypassing
- No CAPTCHA circumvention

### ✅ Privacy Considerations
- Collects business contact info only
- Infers owner names from public sources
- No personal data harvesting
- No social media profile scraping beyond business pages

### ⚠️ User Responsibilities

Before using this workflow:

1. **Review Terms of Service** for each API provider
2. **Comply with local regulations** (GDPR, CCPA, etc.)
3. **Respect opt-out requests** from businesses
4. **Use data ethically** for legitimate business outreach
5. **Maintain data security** for collected information

### API Provider Terms

| Provider | Terms of Service | Rate Limits |
|----------|------------------|-------------|
| Google Places | [ToS](https://cloud.google.com/maps-platform/terms) | 100 QPS |
| Yelp Fusion | [ToS](https://www.yelp.com/developers/api_terms) | 5,000/day |
| SerpAPI | [ToS](https://serpapi.com/terms) | Plan-based |
| Hunter.io | [ToS](https://hunter.io/terms-of-service) | Plan-based |

---

## 📈 Scaling to SaaS

### Phase 1: Multi-Tenant Foundation

```
┌─────────────────────────────────────────────────────────────┐
│                     SaaS Architecture                        │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Frontend   │◀──▶│   Backend    │◀──▶│     n8n      │  │
│  │  Dashboard   │    │     API      │    │   Workflows  │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   │                    │          │
│         ▼                   ▼                    ▼          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   User Auth  │    │   Database   │    │    Queue     │  │
│  │   (Auth0)    │    │  (Postgres)  │    │   (Redis)    │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Key Components:**
- User authentication with Auth0/Clerk
- PostgreSQL for user data and lead storage
- Redis queue for async workflow execution
- REST API for workflow triggering

### Phase 2: Feature Enhancements

1. **Custom Search Campaigns**
   - Users define their own niches/locations
   - Campaign scheduling and history
   - Lead deduplication across campaigns

2. **CRM Integration**
   - Direct push to HubSpot, Salesforce, Pipedrive
   - Automatic contact creation
   - Deal pipeline integration

3. **Outreach Automation**
   - Email sequence templates
   - Personalized AI-generated emails
   - Follow-up scheduling

4. **Analytics Dashboard**
   - Lead conversion tracking
   - ROI measurement
   - Campaign performance metrics

### Phase 3: Enterprise Features

1. **Team Collaboration**
   - Lead assignment and ownership
   - Activity tracking
   - Comments and notes

2. **Advanced AI**
   - Custom scoring models
   - Industry-specific analysis
   - Competitor intelligence

3. **API Access**
   - Webhook integrations
   - Custom data exports
   - Third-party app marketplace

### Pricing Model Example

| Tier | Leads/Month | Features | Price |
|------|-------------|----------|-------|
| Starter | 500 | Basic search, Excel export | $49/mo |
| Pro | 2,000 | AI scoring, CRM integration | $149/mo |
| Business | 10,000 | Multi-user, API access | $399/mo |
| Enterprise | Unlimited | Custom, dedicated support | Custom |

### Tech Stack Recommendations

- **Frontend:** Next.js, Tailwind CSS
- **Backend:** Node.js/Express or Python/FastAPI
- **Database:** PostgreSQL with Prisma
- **Queue:** Redis with BullMQ
- **Auth:** Auth0 or Clerk
- **Hosting:** Vercel + Railway/Render
- **n8n:** Self-hosted or n8n Cloud

---

## 🔧 Troubleshooting

### Common Issues

**1. API Rate Limits Exceeded**
```
Solution: Increase rateLimitDelayMs in configuration
Recommended: 2000-5000ms between requests
```

**2. No Results from Search**
```
Solution: Check API credentials and quotas
Verify search queries are properly formatted
Try different niche/location combinations
```

**3. AI Scoring Fails**
```
Solution: Verify OpenAI/Anthropic API keys
Check API quota and billing
Fallback to rule-based scoring will activate automatically
```

**4. Excel Export Empty**
```
Solution: Ensure leads pass the minimum score filter
Check deduplication isn't removing all leads
Verify enrichment workflow is returning data
```

**5. Website Scraping Blocked**
```
Solution: Some websites block automated access
Consider using SerpAPI for cached results
Add more delay between requests
```

---

## 🤖 n8n-MCP Integration

This workflow system is designed to work seamlessly with the [n8n-MCP](https://github.com/czlonkowski/n8n-mcp) (Model Context Protocol) server. The n8n-MCP provides AI assistants with comprehensive access to n8n node documentation, enabling them to:

- **Understand 1,084+ n8n nodes** (537 core + 547 community)
- **Access node properties and operations** with detailed schemas
- **Reference real-world examples** from 2,646+ workflow templates
- **Validate workflow configurations** before execution
- **Create and modify workflows** programmatically

### Why Use n8n-MCP?

When working with AI assistants to build or modify n8n workflows:

| Without n8n-MCP | With n8n-MCP |
|-----------------|--------------|
| AI has limited knowledge of n8n nodes | AI has access to 87% documentation coverage |
| Manual lookup of node properties | Instant access to detailed schemas |
| Trial-and-error configuration | Validated configurations from templates |
| Generic workflow suggestions | Context-aware recommendations |

### Quick Setup

#### Option 1: Hosted Service (Easiest)

Use the hosted n8n-MCP service at **[dashboard.n8n-mcp.com](https://dashboard.n8n-mcp.com)**:
- ✅ Free tier: 100 tool calls/day
- ✅ No installation required
- ✅ Always up-to-date with latest n8n nodes

#### Option 2: npx (Local)

```bash
# Run directly with npx
npx n8n-mcp
```

Add to Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "http://localhost:5678",
        "N8N_API_KEY": "your-n8n-api-key"
      }
    }
  }
}
```

#### Option 3: Docker

```bash
docker pull ghcr.io/czlonkowski/n8n-mcp:latest
```

Add to Claude Desktop config:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm", "--init",
        "-e", "MCP_MODE=stdio",
        "-e", "LOG_LEVEL=error",
        "-e", "DISABLE_CONSOLE_OUTPUT=true",
        "-e", "N8N_API_URL=http://host.docker.internal:5678",
        "-e", "N8N_API_KEY=your-api-key",
        "ghcr.io/czlonkowski/n8n-mcp:latest"
      ]
    }
  }
}
```

### Using n8n-MCP with This Workflow

Once configured, AI assistants can help you:

1. **Customize the workflows** - Modify search queries, add new data sources
2. **Debug issues** - Understand node configurations and fix errors
3. **Extend functionality** - Add new sub-workflows or integrations
4. **Optimize performance** - Identify bottlenecks and improve efficiency

**Example AI Prompt:**
```
Using n8n-MCP, help me modify the AI Lead Generation workflow to:
1. Add LinkedIn company search as a data source
2. Integrate with HubSpot CRM for lead storage
3. Add a Telegram notification instead of Slack
```

### n8n-MCP Safety Warning

⚠️ **NEVER edit production workflows directly with AI!** Always:
- 🔄 Make a copy of your workflow before using AI tools
- 🧪 Test in development environment first
- 💾 Export backups of important workflows
- ⚡ Validate changes before deploying to production

For more information, see the [n8n-MCP documentation](https://github.com/czlonkowski/n8n-mcp).

---

## 📚 Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community Forum](https://community.n8n.io/)
- [n8n-MCP Server](https://github.com/czlonkowski/n8n-mcp) - AI assistant integration
- [OpenAI API Docs](https://platform.openai.com/docs)
- [Google Places API Docs](https://developers.google.com/maps/documentation/places/web-service)

---

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

---

*Built with ❤️ for the AI automation community*
