# How to Scrape Cadremploi: Executive Jobs in France (Salary, Contract Type, Full Description)

> **Run it now, no code: [cadremploi-jobs-scraper on the Apify Store](https://apify.com/bibian/cadremploi-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 job postings &nbsp;|&nbsp; no login, no API key

Cadremploi is France's leading job board for executives and managers (the "cadres"). It has no public export and no official API for third parties. This guide shows how to scrape Cadremploi executive jobs in France as clean structured data: title, company, city and region, salary range, contract type, publication and expiry dates, and the full description. It uses a free-to-try Apify Actor, from the console or from code, with no login and no API key.

Searches this page answers: Cadremploi scraper, scrape Cadremploi, Cadremploi API, executive jobs France scraper, French executive job postings, cadre job offers data.

## What you get from the Cadremploi scraper

- Executive and management job postings from Cadremploi, newest first, with title, company, city, region and postal code.
- Salary range as numbers (`salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`) when the posting or its listing badge shows one, and `null` otherwise.
- Contract type, normalized (`employmentType`: `full_time`, `temporary`...) and as Cadremploi writes it (`contractLabel`: `CDI`, `CDD`, `Intérim`...).
- Publication date, expiry date, industry, the full description and the candidate profile as plain text.
- Fields named like the other job-board Actors in the same family (schema.org `JobPosting` style), so the output plugs into a pipeline built for several job boards.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results from a run with `keyword: "chef de projet"` and `maxItems: 5` on 17 November 2026, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Stable identifier (`cadremploi:<offer id>`) and the public posting URL |
| `title` | Job title as published |
| `companyName`, `confidentialCompany`, `companyLogoUrl` | Hiring company and its logo; `companyName` is `null` and `confidentialCompany` is `true` when Cadremploi hides the employer |
| `locationLabel`, `city`, `region`, `postalCode`, `country` | Where the job is. `country` is `FR`; some postings give no full city, region or postal code |
| `employmentType` | `full_time`, `part_time`, `contract`, `temporary`, `internship`, `apprenticeship`, `freelance` or `unknown` |
| `contractLabel` | The contract badge as Cadremploi writes it (`CDI`, `CDD`, `Intérim`...) |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range as numbers, `EUR`, and `year` (or `month`, `day`, `hour` when the posting says so); all `null` when no salary is published |
| `salaryLabel` | Cadremploi's own salary badge, for example `50K€ - 55K€` (`null` when there is none) |
| `datePosted`, `validThrough` | Publication and expiry dates, ISO 8601 |
| `industry`, `occupationalCategory` | Sector as named by the site |
| `description` | Full posting text, with personal contact details removed |
| `qualifications` | The candidate profile part of the posting, when the site separates it |
| `directApply` | `true` when the posting accepts applications directly |
| `remote`, `experienceLevel`, `skills`, `companyUrl` | Kept for the shared schema; `remote` is `unknown` unless the posting declares remote work, `experienceLevel` and `companyUrl` are `null` and `skills` is `[]` when the site gives none |
| `source`, `sourceId`, `scrapedAt` | `cadremploi.fr`, the native offer id and the extraction timestamp |

## Use cases

- **Monitor executive hiring.** See which companies recruit for a function, a title or a region, and how that changes week to week.
- **Feed a daily job alert.** Run it every morning with a keyword and `onlyNew`, and send only the new postings to Slack, e-mail or a Google Sheet.
- **Benchmark pay.** Compare salary ranges across companies for the same kind of role, in the postings that publish one.
- **Build a job aggregator.** Cadremploi offers no export for third parties; this Actor reads the same postings its own search page shows.

## Input

| Field | What it does |
|---|---|
| `keyword` | Job title, skill or function, exactly as typed in Cadremploi's own search box, for example `chef de projet` or `acheteur informatique`. Empty means every executive posting. |
| `maxItems` | The run stops once this many postings are saved (1 to 2000, default 30). |
| `onlyNew` | For scheduled runs. Skip the postings an earlier run of this same search already delivered, so each run returns, and charges, only what is new (default off). |

Minimal input, the 30 newest executive postings, whatever the function:

```json
{}
```

Executive project managers, 100 postings (the input used by the examples below):

```json
{
  "keyword": "chef de projet",
  "maxItems": 100
}
```

Daily watch for a function, charged only for what is new:

```json
{
  "keyword": "acheteur informatique",
  "onlyNew": true,
  "maxItems": 200
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/cadremploi-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Type a keyword such as `chef de projet`, set the maximum number of postings, then click **Start**.
3. When the run finishes, open the **Output** tab, check the table (title, company, city, region, contract type, salary, date, link), then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor, tick **Only new results**, and optionally add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/cadremploi-jobs-scraper').call({
    keyword: 'chef de projet',
    maxItems: 100,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName ?? '-'} | ${item.city ?? '-'} | ${item.contractLabel ?? '-'} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryCurrency ?? ''}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/cadremploi-jobs-scraper").call(run_input={
    "keyword": "chef de projet",
    "maxItems": 100,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('city')} | {item.get('contractLabel')} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryCurrency') or ''}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$2.00 per 1,000 job postings** (that is $0.002 per posting), plus a start fee of $0.0025 per GB of run memory per run (the Actor runs on 4 GB by default, so about $0.01 per run). You pay only for the results saved to the dataset. These are the free-plan prices; paid Apify plans get lower ones. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/cadremploi-jobs-scraper) when this guide was written (17 November 2026); the Store page is the reference.

## Limits and honest notes

- **It is slower and costs more than a typical job-board scraper.** Cadremploi puts a managed Cloudflare challenge on every page, so the Actor drives a real stealth browser session for each one, listing or posting. Postings are read one at a time: a run of 30 postings takes about two minutes, and a long export needs a higher run timeout in the Run options (the default is one hour). Postings are saved in groups of 10 as they are read, so a run stopped early keeps everything read so far.
- **Without a keyword, one agency can dominate the results.** Newest postings come first, and in a run on 17 November 2026 with no keyword, 29 of the first 30 postings were relayed by the same staffing company. Use `keyword` to target a function.
- **Salary is only there when the site publishes it.** It is read from the posting's structured data, otherwise from the listing's `NNK€ - NNK€` badge. A plain `NN€ - NN€` badge is left out rather than guessed, since nothing says whether it is a daily or an hourly rate. In the run mentioned above, 29 of 30 postings had a salary, and 4 of those 29 held figures between 3,300 and 4,500 with `salaryPeriod` set to `year`: they look like monthly amounts, so read `salaryLabel` too and treat values far below annual levels with care. In the keyword run behind `sample-output.json`, 2 of 5 postings had a salary.
- **Location depends on the posting.** It is read from the posting's own address fields; not every posting gives a full city, region and postal code.
- **A posting relayed by a partner board shows that partner as the company** (for example `Free-Work` in the sample), as published by Cadremploi itself.
- The dataset contains job postings only. Only the recruiting company's own name, logo and text are kept; any phone number or personal e-mail address found in a posting's text is removed before saving.
- This is an unofficial tool, not affiliated with Cadremploi. Check the site terms and the data protection rules that apply to your use, for example the GDPR.

## FAQ

**Does it need a login, an API key or a proxy on my side?**  
No. It reads the publicly visible postings. The browser session Cadremploi requires runs inside the Actor.

**Can I filter by job title or function?**  
Yes, with the `keyword` input field. It searches the same way as Cadremploi's own search box.

**How do I only get new postings every day?**  
Tick `onlyNew` and schedule the Actor once a day with the same keyword. Each run then returns, and charges, only the postings that earlier runs of the same search did not already deliver.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

**Does it collect any individual's personal data?**  
No. Only company names, logos and descriptions are kept; phone numbers and personal e-mail addresses found in a posting's text are removed before saving.

**Is it legal to scrape?**  
The Actor collects publicly visible postings. How you use the data is your responsibility: check the site's terms and the laws that apply to you, for example the GDPR.

## En français

Cet extracteur récupère les offres cadres publiées sur Cadremploi : intitulé, entreprise, lieu, salaire, type de contrat, dates et description complète, en JSON, CSV ou Excel. Aucun compte Cadremploi ni clé d'API n'est nécessaire, et aucune donnée personnelle n'est conservée.

## Try it

**[Open cadremploi-jobs-scraper on the Apify Store](https://apify.com/bibian/cadremploi-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
