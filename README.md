# Slovenian News Dataset – Russo-Ukrainian War Period (RTV Slovenija, 2022–2025)

A corpus of **9,869 Slovene news articles** collected from the public news portal of the
Slovenian national broadcaster, [rtvslo.si](https://www.rtvslo.si), covering
**February 2022 – June 2025**, i.e. the period from the Russian full-scale invasion of
Ukraine onwards. It was built for the master's thesis *Semantic Analysis of Media in
Relation to the Russo-Ukrainian War* (University of Ljubljana, Faculty of Computer and
Information Science, 2026) and is released so that the analyses can be reproduced and the
corpus reused.

The code that produced it (crawler + analysis pipeline) lives in
[toti2026/thesis_code](https://github.com/toti2026/thesis_code).

## Contents

```
dataset.csv   9,869 rows × 7 columns, UTF-8, comma-separated (≈ 49 MB)
README.md
```

### Columns

| Column       | Type   | Description |
|--------------|--------|-------------|
| `article_id` | int    | Numeric article identifier taken from the rtvslo.si URL (unique). The article URL is `https://www.rtvslo.si/<section>/.../<article_id>`. |
| `title`      | str    | Article headline. |
| `subtitle`   | str    | Sub-headline shown under the title (empty for ~9 % of articles). |
| `slug`       | str    | **Lead paragraph** (the teaser shown in bold under the headline). The column name is historical – it is *not* a URL slug. |
| `date`       | str    | Publication date as `DD-MM-YYYY`, parsed from the article's publication metadata. **See the note on month `13` below.** |
| `tags`       | str    | Editorial tags assigned by rtvslo.si, separated by `; ` (e.g. `Ukrajina; Rusija; Nato`). On average 3.9 tags per article, 13,445 distinct tags. |
| `content`    | str    | Full article body as plain text (all `<p>` elements concatenated). |

### Size and coverage

| | |
|---|---|
| Articles | 9,869 |
| Publication period | 2022-02-01 → 2025-06-29 |
| Articles per year | 2022: 4,132 · 2023: 1,753 · 2024: 1,907 · 2025: 992 (plus 709 December articles, see below) |
| Mean / median article length | 680 / 458 words |
| Most frequent tags | Ukrajina (2,110), Rusija (1,784), ZDA (386), vojna (371), Izrael (350), Slovenija (260), EU (243), Gaza (195), Vladimir Putin (190), Nato (169) |

## How the data was collected

1. **URL discovery** – `slocrawler.py` queries the rtvslo.si site search in quarterly date
   windows from 2022-02-01 to 2025-06-30 and collects the article URLs from every
   result page.
2. **Article download** – `sloArticleCrawler.py` fetches each URL and extracts title,
   subtitle, lead, publication date, tags and body text from the article HTML.
   Requests are rate-limited (3.5 s between articles, 5 s between batches).
3. Results were merged into `dataset.csv`; exact duplicates (same `article_id`) were
   removed.

The corpus is **not pre-filtered for war relevance**: it is everything the site search
returned for the query terms in the given period. About 21 % of articles carry the tag
*Ukrajina* and a larger share mention Ukraine, Russia or the war in the text, but the
corpus also contains unrelated domestic news (COVID-19, Eurovision, elections, sports).
In the thesis, war-related articles are isolated with topic modeling (BERTopic);
the `tags` column is a quick alternative filter.

## Known issues
* **Missing dates** – 376 rows have an empty `date` (the publication metadata block
  was absent or in an unexpected format).
* **Missing fields** – `subtitle` empty in 923 rows, `tags` in 400, `slug` in 25,
  `content` in 18.
* **Near-duplicates** – 95 repeated titles and 30 repeated bodies (live blogs and
  updated articles published under new IDs). Deduplicate on `content` if this matters
  for your use case.

## Loading the data

```python
import pandas as pd

df = pd.read_csv("dataset.csv")

# fix the December encoding and parse dates (DD-MM-YYYY)
df["date"] = df["date"].str.replace(r"^(\d{1,2})-13-(\d{4})$", r"\1-12-\2", regex=True)
df["date"] = pd.to_datetime(df["date"], format="%d-%m-%Y", errors="coerce")

# tags as lists
df["tags"] = df["tags"].fillna("").apply(lambda s: [t.strip() for t in s.split(";") if t.strip()])

# example: war-related subset by tag
war = df[df["tags"].apply(lambda ts: any(t in ts for t in ("Ukrajina", "Rusija", "vojna v Ukrajini")))]
```

## License and terms of use

The article texts are © RTV Slovenija and remain subject to the portal's terms of use.
They are redistributed here **for non-commercial research and educational purposes
only**, in the form in which they were publicly accessible at collection time. The
compilation (selection, cleaning, metadata) is released under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). If you are RTV Slovenija and
have concerns about this redistribution, please open an issue.

## Citation

If you use this dataset, please cite the thesis:

```bibtex
@mastersthesis{kadriu2026semantic,
  author  = {Kadriu, Kastriot},
  title   = {Semantic analysis of media in relation to the {Russo-Ukrainian} war},
  school  = {University of Ljubljana, Faculty of Computer and Information Science},
  address = {Ljubljana, Slovenia},
  year    = {2026},
  note    = {Dataset: \url{https://github.com/toti2026/slovenian_news_dataset}}
}
```

## Related

* Analysis code and crawler: <https://github.com/toti2026/thesis_code>
