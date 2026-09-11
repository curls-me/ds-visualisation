# Cheatsheet: Notebooks 2 & 3

Line-by-line code reference for **`2_Plotting_intro.ipynb`** (Matplotlib / Seaborn / Plotly)
and **`3_Fetching_the_data.ipynb`** (psycopg2 / SQLAlchemy).

---

## Part 1 — Notebook 2: Plotting Intro

### 1.1 Imports & data loading

```python
import pandas as pd                # data handling: DataFrames
import matplotlib.pyplot as plt    # low-level plotting engine
import seaborn as sns              # statistical plots, built on matplotlib
import plotly.express as px        # high-level interactive plots
import plotly.graph_objects as go  # low-level interactive plots (full control)

df = pd.read_csv("data/penguins_clean.csv")  # load the dataset from disk into a DataFrame
print(f"Shape: {df.shape}")                  # (rows, columns) — quick sanity check
df.head()                                    # preview the first 5 rows

df.info()                 # column names, data types, non-null counts
df.describe().round(2)    # count/mean/std/min/max/quartiles for numeric columns, rounded to 2dp

missing = df.isna().sum()          # count missing (NaN) values per column
print(missing[missing > 0])        # only print columns that actually have missing values
```

---

### 1.2 Matplotlib

**Core concept:** `Figure` = the whole canvas. `Axes` = one plot area inside it. `fig, ax = plt.subplots()` gives you both as objects you call methods on.

**Line plot**

```python
df_binned = df.copy()                                              # copy so we don't alter the original df
df_binned["mass_bin"] = pd.cut(df["body_mass_g"], bins=10)         # split body mass into 10 equal-width bins
avg_flipper = df_binned.groupby("mass_bin", observed=True)["flipper_length_mm"].mean()
                                                                     # mean flipper length per bin

fig, ax = plt.subplots(figsize=(10, 5))   # create figure (10x5 inches) and one Axes

ax.plot(
    range(len(avg_flipper)),   # x: bin index (0, 1, 2, ...)
    avg_flipper.values,        # y: the mean flipper length per bin
    color="steelblue",         # line colour
    linewidth=2.5,             # line thickness
    marker="X",                # marker shape drawn at each data point
    markersize=8,               # marker size in points
    label="Mean flipper length",  # text shown in the legend
)

ax.set_title("...", fontsize=14, fontweight="bold")  # chart title
ax.set_xlabel("...", fontsize=12)                     # x-axis label
ax.set_ylabel("...", fontsize=12)                     # y-axis label
ax.legend()                                           # draw the legend using the label= values above
ax.grid(True, linestyle="--", alpha=0.5)              # dashed, semi-transparent gridlines

plt.tight_layout()  # auto-adjust spacing so labels/titles don't get clipped
plt.show()           # render the figure
```

**Scatter plot**

```python
species_colours = {                # manual colour map so each species gets a consistent colour
    "Adelie": "#E07B54",
    "Chinstrap": "#5B9BD5",
    "Gentoo": "#70AD47",
}

fig, ax = plt.subplots(figsize=(10, 7))

for species, colour in species_colours.items():        # loop once per species so each gets its own legend entry
    subset = df[df["species"] == species]              # rows belonging to this species only
    ax.scatter(
        subset["bill_length_mm"],   # x values
        subset["bill_depth_mm"],    # y values
        c=colour,                   # dot colour
        label=species,              # legend text
        alpha=0.75,                 # transparency (0=invisible, 1=solid) — helps see overlap
        edgecolors="white",         # outline colour around each dot
        linewidths=0.5,             # outline thickness
        s=70,                       # dot size
    )

ax.legend(title="Species", fontsize=11)  # legend built from the label= of each scatter() call
```

**Bar chart**

```python
avg_mass = df.groupby("species")["body_mass_g"].mean().sort_values(ascending=False)
                                        # mean body mass per species, largest first

bar_colours = [
    highlight_colour if sp == "Gentoo" else default_colour for sp in avg_mass.index
]                                       # colour list: highlight one category, grey out the rest

fig, ax = plt.subplots(figsize=(8, 6))
bars = ax.bar(
    avg_mass.index, avg_mass.values,   # x = species names, y = mean mass
    color=bar_colours, edgecolor="white", linewidth=1.5
)

for bar in bars:                        # loop over each bar to label it individually
    height = bar.get_height()           # the bar's y-value (the mean mass)
    ax.text(
        bar.get_x() + bar.get_width() / 2,  # x position: horizontal centre of the bar
        height + 30,                        # y position: just above the bar top
        f"{height:,.0f} g",                 # text: value formatted with thousands separator
        ha="center", va="bottom", fontsize=11,  # horizontal/vertical text alignment
    )

ax.annotate(
    "Gentoo are ~900g heavier...",      # annotation text
    xy=(0, avg_mass["Gentoo"]),          # point the arrow is pointing AT
    xytext=(1.5, avg_mass["Gentoo"] - 200),  # where the text itself sits
    arrowprops=dict(arrowstyle="->", color="grey"),  # draws the connecting arrow
)

ax.set_ylim(0, avg_mass.max() * 1.2)     # y-axis range, with headroom for the value labels
ax.grid(axis="y", linestyle="--", alpha=0.4)  # gridlines on the y-axis only
ax.spines["top"].set_visible(False)      # hide the top border of the plot box
ax.spines["right"].set_visible(False)    # hide the right border ("open" look)
```

**Histogram**

```python
df_male = df[df["sex"] == "male"]      # subset: male rows only
df_female = df[df["sex"] == "female"]  # subset: female rows only

fig, ax = plt.subplots(figsize=(10, 6))

ax.hist(
    df_male["body_mass_g"],   # values to bin
    bins=20,                  # number of bins
    alpha=0.6,                # transparency, so the two overlapping histograms are both visible
    color="steelblue", label="Male", edgecolor="white",
)
ax.hist(df_female["body_mass_g"], bins=20, alpha=0.6, color="salmon", label="Female", edgecolor="white")

ax.axvline(
    df_male["body_mass_g"].mean(),   # x position: the mean value
    color="steelblue", linestyle="--", linewidth=2,
    label=f"Male mean: {df_male['body_mass_g'].mean():.0f}g",  # vertical reference line at the mean
)
```
`ax.axvline(x=...)` draws a **vertical** line at that x value; `ax.axhline(y=...)` is the mirror — a **horizontal** line at that y value.

**Subplots (multiple Axes in one Figure)**

```python
islands = df["island"].unique()     # the distinct island names
n_islands = len(islands)            # how many subplots we need

fig, axes = plt.subplots(nrows=1, ncols=n_islands, figsize=(14, 6), sharey=True)
                                     # 1 row, N columns of Axes; sharey=True keeps y-axis scale identical across panels

for ax, island, colour in zip(axes, islands, colours):  # loop over each subplot Axes + matching island + colour
    island_data = df[df["island"] == island]["body_mass_g"].dropna()  # values for this island, NaNs removed

    bp = ax.boxplot(
        island_data,
        patch_artist=True,   # allows the box to be filled with a colour (not just an outline)
        notch=False,          # no notch narrowing at the median
        widths=0.5,           # box width
    )
    bp["boxes"][0].set_facecolor(colour)  # colour the box fill
    ax.set_xticks([])                     # hide x tick marks (only one category per subplot, title says which)

fig.suptitle("...", y=1.02)   # one title for the whole figure (not per-subplot)
```

**Style sheets**

```python
print(plt.style.available)   # list every built-in theme name

with plt.style.context(style):   # temporarily apply a style, only inside this `with` block
    ax.scatter(...)
```

---

### 1.3 Seaborn

Two API families: **axes-level** functions (`scatterplot`, `histplot`, `boxplot`, `violinplot`, `heatmap`) take `ax=` and draw into an existing Matplotlib Axes; **figure-level** functions (`relplot`, `displot`, `catplot`, `lmplot`, `pairplot`) build their own multi-panel figure and don't take `ax=`.

**Scatter with automatic legend**

```python
sns.scatterplot(
    data=df, x="bill_length_mm", y="flipper_length_mm",
    hue="species",   # colour points by species (auto-creates a colour legend)
    style="sex",     # vary marker shape by sex (auto-creates a shape legend)
    palette="Set2",  # named colour palette
    alpha=0.8, s=80, # transparency, marker size
    ax=ax,           # draw into this existing Axes (axes-level function)
)
```

**Regression plot**

```python
sns.lmplot(
    data=df, x="bill_length_mm", y="flipper_length_mm",
    hue="species",          # separate regression line + scatter per species
    height=6, aspect=1.3,   # figure size controls (figure-level function, no ax=)
    scatter_kws={"alpha": 0.5, "s": 40},  # extra styling passed through to the underlying scatter
)
# The shaded band around each line = 95% confidence interval, computed automatically.
```

**Distribution plot with KDE**

```python
sns.histplot(
    data=df, x=col,
    hue="species",  # one histogram colour per species
    kde=True,       # overlay a smoothed density curve on top of the bars
    palette="Set2", alpha=0.5, ax=ax,
)
```

**Box vs. violin**

```python
sns.boxplot(data=df, x="species", y="body_mass_g", hue="sex", palette="pastel", ax=axes[0])
# Box = median, IQR (25th-75th percentile box), whiskers, outlier dots.

sns.violinplot(
    data=df, x="species", y="body_mass_g", hue="sex",
    split=True,   # draw each hue level as one half of the violin, instead of side-by-side violins
    palette="pastel", ax=axes[1],
)
# Violin = same info as a box plot, plus the full shape of the distribution (bimodal? skewed?).
```

**catplot (figure-level, multi-panel categorical)**

```python
g = sns.catplot(
    data=df, x="island", y="body_mass_g", hue="species",
    kind="box",      # which categorical plot type to draw: box/violin/strip/swarm
    height=5, aspect=1.5,
)
sns.move_legend(g, "upper right", bbox_to_anchor=(0.98, 0.98), title="Species")
                     # reposition seaborn's default legend on a figure-level plot
```

**Correlation heatmap**

```python
numeric_df = df.select_dtypes(include="number")     # keep only numeric columns
corr_matrix = numeric_df.corr(method="pearson").round(2)  # pairwise correlation matrix, 2dp

mask = np.triu(np.ones_like(corr_matrix, dtype=bool))  # boolean mask for the upper triangle (it mirrors the lower one)

sns.heatmap(
    corr_matrix,
    mask=mask,          # hide the cells where mask is True
    annot=True,          # print the numeric value inside each cell
    fmt=".2f",           # number format for the annotations
    cmap="coolwarm",     # colour scale: red = positive, blue = negative
    vmin=-1, vmax=1,     # fix the colour scale to the full correlation range
    square=True,         # force each cell to be square
    linewidths=0.5,      # thin white lines between cells
    ax=ax,
)
```

**Pairplot (grid of every variable pair)**

```python
g = sns.pairplot(
    data=df, hue="species",
    diag_kind="kde",   # diagonal cells show a density curve instead of a histogram
    plot_kws={"alpha": 0.5, "s": 30},  # styling passed to the off-diagonal scatter plots
)
```

**relplot (facet grid by two categorical variables)**

```python
g = sns.relplot(
    data=df, x="bill_length_mm", y="bill_depth_mm", hue="island",
    col="species",   # one column of subplots per species
    row="sex",       # one row of subplots per sex
    height=3.5, aspect=1,
)
```

---

### 1.4 Plotly

Two APIs: `plotly.express` (`px`) — high-level, one call builds a full chart; `plotly.graph_objects` (`go`) — low-level, build trace by trace for full control.

**Interactive scatter**

```python
fig = px.scatter(
    data_frame=df, x="bill_length_mm", y="bill_depth_mm",
    color="species",   # colour by species
    symbol="sex",      # marker shape by sex
    hover_data=["island", "body_mass_g", "flipper_length_mm"],  # extra columns shown on hover
    title="...",
    labels={...},        # rename axis/legend text away from raw column names
    template="plotly_white",  # visual theme
    width=800, height=550,
)
fig.show()   # render as an interactive HTML widget
```

**Grouped bar chart**

```python
count_df = df.groupby(["island", "species"]).size().reset_index(name="count")
                        # count of rows per island+species combo, as a tidy DataFrame

fig = px.bar(
    count_df, x="island", y="count", color="species",
    barmode="group",  # bars for each species sit side by side ('stack' would pile them into one bar)
    color_discrete_map={...},  # fixed colour per category, instead of an auto-assigned palette
)
fig.update_layout(legend_title_text="Species")  # edit a layout property after the figure is built
```

**Violin plot with embedded box**

```python
fig = px.violin(
    df, x="species", y="body_mass_g", color="sex",
    box=True,             # draw a mini box plot inside each violin
    points="outliers",    # only show individual points that are outliers ('all' or False are other options)
    hover_data=df.columns,  # show every column on hover
)
```

**Heatmap with graph_objects (lower-level API)**

```python
fig = go.Figure(
    data=go.Heatmap(
        z=corr.values,              # the matrix of values to colour
        x=corr.columns.tolist(),    # column labels
        y=corr.index.tolist(),      # row labels
        colorscale="RdBu_r",        # red = positive, blue = negative
        zmin=-1, zmax=1,            # fix colour scale range
        text=corr.values, texttemplate="%{text}",  # print the value inside each cell
    )
)
fig.update_layout(title="...", width=600, height=500)
```

**Choropleth map**

```python
with urlopen(GEO_JSON_URL) as response:
    counties = json.load(response)      # load county boundary shapes (GeoJSON format)

unemp_df = pd.read_csv(url, dtype={"fips": str})  # fips kept as string so leading zeros aren't dropped

fig = px.choropleth(
    data_frame=unemp_df,
    geojson=counties,       # the shape boundaries to draw
    locations="fips",       # column that matches each row to a shape via its id
    color="unemp",          # column that sets each region's fill colour
    color_continuous_scale="Reds",
    range_color=(0, 12),    # fix the colour scale range
    scope="usa",            # crop the map to just the USA
)
```

**Animation**

```python
fig = px.scatter(
    gapminder,
    x="gdpPercap", y="lifeExp",
    animation_frame="year",     # one animation frame per year, adds a play button + slider
    animation_group="country",  # tracks each country as the same point across frames
    size="pop",                 # bubble size encodes population
    log_x=True,                 # log scale on x (GDP spans several orders of magnitude)
    size_max=60,                # caps the largest bubble size
)
```

**Saving an interactive chart**

```python
fig_to_save.write_html("assets/penguin_scatter_interactive.html")
# Produces a standalone HTML file — anyone can open it in a browser, no Python needed.
# fig_to_save.write_image(...) would instead save a static PNG (requires the `kaleido` package).
```

---

### 1.5 Library decision guide

| Question | Answer |
|---|---|
| Audience needs to interact (hover/zoom/filter)? | **Plotly** |
| Need automatic stats (regression, CI, KDE, correlation)? | **Seaborn** |
| Quick EDA vs. final polished output? | EDA → Seaborn · Publication → Matplotlib · Web report → Plotly |
| Need unusual/custom chart types? | **Matplotlib** |

---

## Part 2 — Notebook 3: Fetching Data from SQL

### 2.1 Why connect from Python

Manual workflow (Python → CSV → DBeaver → SQL client → CSV → Python) takes 4 steps across 2 tools. Connecting directly from Python cuts it to a single step: query the database, get a DataFrame back.

### 2.2 Approach 1 — `psycopg2` (low-level PostgreSQL driver)

```python
import pandas as pd              # turn SQL results into DataFrames
import psycopg2                  # PostgreSQL driver, follows the DB-API 2.0 standard
import os                        # read environment variables
from dotenv import load_dotenv   # load variables from a local .env file
```

**Credentials via `.env`** (never hardcode secrets in a notebook — they'd end up on GitHub)

```python
load_dotenv()   # reads the .env file in the project root, injects its values as environment variables

DATABASE = os.getenv("DATABASE")   # each os.getenv() reads one variable set in .env
USER_DB  = os.getenv("USER_DB")
PASSWORD = os.getenv("PASSWORD")
HOST     = os.getenv("HOST")
PORT     = os.getenv("PORT")
```

**Connection + cursor** (a *connection* is the open session with the server; a *cursor* is the object you use to send queries through it)

```python
conn = psycopg2.connect(
    database=DATABASE, user=USER_DB, password=PASSWORD, host=HOST, port=PORT
)   # opens the session

cur = conn.cursor()                                       # create a cursor bound to this connection
cur.execute("SELECT * FROM datasets.kaggle_survey LIMIT 5")  # send the SQL statement
rows = cur.fetchall()                                      # pull all matching rows back as a list of tuples

conn.close()   # always release the connection when you're done with it
```

**Loading straight into a DataFrame (skips manual cursor handling)**

```python
conn = psycopg2.connect(database=DATABASE, user=USER_DB, password=PASSWORD, host=HOST, port=PORT)

query = "SELECT * FROM datasets.kaggle_survey LIMIT 10"
df_psycopg = pd.read_sql(query, conn)   # runs the query and returns a DataFrame directly — no fetchall() needed

conn.close()
```

### 2.3 Approach 2 — `SQLAlchemy` (higher-level, multi-database abstraction)

```python
from sqlalchemy import create_engine   # SQLAlchemy's connection factory

load_dotenv()
DB_STRING = os.getenv("DB_STRING")     # a single connection string encodes user/password/host/db in one value

db = create_engine(DB_STRING)          # the engine manages connections for you; same API works for Postgres/MySQL/SQLite/etc.

query = "SELECT * FROM datasets.kaggle_survey"     # no LIMIT this time — pull the full table
df_sqlalchemy = pd.read_sql(query, db)             # query straight into a DataFrame using the engine
```

### 2.4 Export to CSV (so later notebooks don't need to re-query)

```python
df_sqlalchemy.to_csv(
    "data/kaggle_survey.csv",
    index=False,   # don't write pandas' row-number index as an extra column
)
```

### 2.5 Key takeaways

- `psycopg2` = PostgreSQL-only, lower-level (manual cursor). `SQLAlchemy` = works across many database engines via one connection-string API.
- Both can hand results straight to `pd.read_sql(query, conn_or_engine)` — no manual row-by-row parsing needed.
- Credentials always live in a git-ignored `.env` file, read at runtime with `os.getenv()` — never typed directly into a notebook.
- Export once to CSV so downstream notebooks load instantly, don't need live DB credentials, and stay reproducible (the exact dataset is frozen on disk).

---

## Part 3 — Applied Example: Exercise Solution (`solutions/4_Visualization_exercise.ipynb`)

This is the worked solution that puts notebooks 2 and 3 into practice on the real Kaggle survey dataset. It shows the pattern you'll reuse most often: **clean → aggregate → plot with a finding-style title.**

**Data cleaning: turning a text salary range into a number**

```python
compensation = df[["latest_job_role", "yearly_earnings"]].copy()  # keep only the columns needed, .copy() avoids a pandas warning
compensation = compensation.dropna()                              # drop rows with no salary answer

def get_first_number(x):
    """Extract the lower bound from a salary range string."""
    x = x.split("-")[0]                                    # keep only the text before the '-', e.g. "$10,000-$14,999" -> "$10,000"
    x = x.replace(",", "").replace(">", "").replace("$", "").strip()  # strip everything that isn't a digit
    return int(x)                                           # convert the cleaned string to an integer

compensation["salary_usd"] = compensation["yearly_earnings"].apply(get_first_number)
                                                             # apply() runs the function once per row, building a new numeric column

roles_of_interest = ["Data Scientist", "Data Analyst", "Data Engineer"]  # the three roles the stakeholder asked about
data_compensation = compensation[
    compensation["latest_job_role"].isin(roles_of_interest)  # keep only rows whose role is in that list
]
```

**Explanatory box plot (answers "compare compensation across roles")**

```python
plt.figure(figsize=(9, 6))            # shorthand for fig, ax when you don't need the ax object afterwards
sns.boxplot(
    data=data_compensation, x="latest_job_role", y="salary_usd",
    hue="latest_job_role",   # colour each box by role
    palette="Set2",
    legend=False,            # hide the legend: it would just repeat the x-axis labels
)
plt.title("Yearly Earnings Distribution: Data Scientists, Analysts, and Engineers")  # states what the chart shows, not just the variable names
```

**Two related bar charts from one `value_counts()` call**

```python
top_10_countries = df["county_residence"].value_counts().head(10).index.tolist()
                        # count respondents per country, sort descending, keep the top 10 country names as a list
df_top10 = df[df["county_residence"].isin(top_10_countries)].copy()  # filter the full dataset down to just those countries

counts_2a = (
    df_top10.groupby(["county_residence", "gender"]).size().reset_index(name="count")
)                       # one row per country+gender combination, with a count column

fig = px.bar(
    counts_2a, x="county_residence", y="count", color="gender",
    barmode="stack",                                  # stack genders within each country bar (shows the country total)
    category_orders={"county_residence": top_10_countries},  # keep countries in "most respondents first" order, not alphabetical
)
fig.update_layout(xaxis_tickangle=-30)  # tilt long country names so they don't overlap

women = df[df["gender"] == "Woman"]                              # filter to one gender
top_women = women["county_residence"].value_counts().head(10).reset_index()  # count by country, keep top 10
top_women.columns = ["country", "count"]                         # value_counts()'s default column names aren't descriptive, so rename them

fig = px.bar(
    top_women, x="count", y="country",
    orientation="h",                       # horizontal: keeps long country names readable on the y-axis
    color="count",                          # colour intensity doubles as a second encoding of the same value
    color_continuous_scale="Blues",
)
fig.update_layout(yaxis={"categoryorder": "total ascending"})  # sort bars by value instead of default alphabetical/insertion order
```

**Grouped comparison across categories**

```python
no_python = df[df["programming_language_recommended"] != "Python"].copy()  # exclude the known top answer to see what's #2
lang_counts = no_python["programming_language_recommended"].value_counts().reset_index()
lang_counts.columns = ["language", "count"]

role_lang = (
    df_roles.groupby(["latest_job_role", "programming_language_recommended"])
    .size()
    .reset_index(name="count")
)                       # one row per role+language combination

fig = px.bar(
    role_lang, x="programming_language_recommended", y="count",
    color="latest_job_role",
    barmode="group",     # 'group' puts roles side by side per language, so you can compare them directly (vs. 'stack' for totals)
)
```

**Quick exploratory chart (Matplotlib, testing a one-line hypothesis)**

```python
education_counts = df["highest_education"].value_counts().sort_values()  # counts per level, ascending so the largest bar is at the top of a barh

fig, ax = plt.subplots(figsize=(10, 5))
ax.barh(education_counts.index, education_counts.values, color="#5B9BD5", edgecolor="white")  # barh = horizontal bar chart
ax.spines[["top", "right"]].set_visible(False)  # remove two of the four box borders for a cleaner look
```

**Takeaway pattern:** every plot in this exercise follows the same shape — `value_counts()`/`groupby()` to aggregate → filter to the top N or the categories that matter → one plot call → a title that states the finding, not just the variable names.

---

## Guidelines for LLMs Writing Visualization Code

If you (an LLM/AI assistant) are asked to generate plotting or data-prep code based on this cheatsheet, follow these to keep the output as simple as possible:

1. **Pick the simplest chart that answers the question.** A bar chart or box plot beats a custom multi-trace figure unless the stakeholder question genuinely needs more (e.g. interactivity, geography, animation — see the decision guide in Part 1.5).
2. **Use library defaults first.** Only add custom colours, annotations, or styling when they serve the specific point being made (e.g. highlighting one bar, as in the body-mass example) — not by default on every chart.
3. **One aggregation, one plot.** Prefer `value_counts()` / `groupby().size()` / `.mean()` directly in the plotting cell over building intermediate DataFrames or classes you don't reuse elsewhere.
4. **Only write a helper function if the transform repeats.** `get_first_number()` earns its place because it's applied per-row via `.apply()`; don't wrap a one-off `groupby` in a function "for readability."
5. **Titles state the finding, not the variable names.** Weak: `"Salary by Job Role"`. Strong: `"Data Engineers Tend to Earn More Than Data Analysts"`. Only relax this for quick exploratory charts, which don't need to be presentation-ready.
6. **Don't add error handling or validation for data shapes that can't occur** in a one-off notebook analysis (e.g. don't defensively check that a column exists before using it) — that belongs in production pipelines, not exploratory/explanatory notebooks.
7. **Match the library to the actual need**, per the decision guide: Matplotlib for full control/publication, Seaborn for automatic statistics, Plotly only when interactivity, maps, or animation add real value — not as a default choice.
8. **Comment the *why*, not the *what*.** Good: `# barmode='group' so roles are directly comparable, not stacked totals`. Skip comments that just restate the function name.
