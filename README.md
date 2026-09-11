# Data Visualizations with Python

A hands-on module on data visualization in Python. You will work through Matplotlib, Seaborn, and Plotly, learn when to reach for exploratory versus explanatory charts, pull data from a SQL database, and build a stakeholder-ready set of plots from the 2020 Kaggle Data Science Survey. The Palmer Penguins dataset is used for the early lessons.

## Learning Objectives

By the end of this repository, you should be able to:

- Create visualizations with Matplotlib, Seaborn, and Plotly.
- Explain when to use exploratory versus explanatory visualizations.
- Fetch data from a SQL database and prepare it for plotting.
- Choose the right chart type for a given data type and question.
- Polish a chart into a clear, audience-ready explanatory visualization.

## Learning Path

The lessons build on each other in order:

| File / Folder | Description |
|---|---|
| [**1 - Exploratory vs Explanatory**](1_Exploratory_vs_explanatory_viz.md) | Reading on the two purposes of visualization and the explore, refine, polish workflow. |
| [**2 - Plotting Intro**](2_Plotting_intro.ipynb) | A tour of Matplotlib, Seaborn, and Plotly with diverse chart types and checkpoints, using the Palmer Penguins dataset. |
| [**3 - Fetching the Data**](3_Fetching_the_data.ipynb) | Connect to a PostgreSQL database with `psycopg2` and `SQLAlchemy`, query it, and export to CSV. |
| [**4 - Visualization Exercise**](4_Visualization_exercise.ipynb) | The main assignment: a stakeholder-driven challenge on the 2020 Kaggle survey where you build at least one plot each with Matplotlib, Seaborn, and Plotly. |
| [**5 - Fantastic Charts**](5_Fantastic_charts.ipynb) | A worked gallery from minimal to publication-styled charts across all three libraries, with a quick-reference appendix. |

### Additional Folders and Files

| File / Folder | Description |
|---|---|
| [**Data**](data/) | Datasets used by the notebooks (for example, `penguins_clean.csv`). |
| [**Assets**](assets/) | Supporting images and exported charts for the lessons. |
| [**Solutions**](solutions/) | Worked solutions to the exercises in the notebooks. |
| [**Kaggle Survey 2020 Methodology**](kaggle_survey_2020_methodology.pdf) | Methodology for the 2020 Kaggle survey dataset. |
| [**Kaggle Survey 2020 Answer Choices**](kaggle_survey_2020_answer_choices.pdf) | Answer choices reference for the Kaggle survey questions. |
| [**pyproject.toml**](pyproject.toml) | Project configuration and dependencies. |
| [**uv.lock**](uv.lock) | Dependency lock file. |

## Setup

> [!NOTE]
> Throughout these steps, text in angle brackets like `<repo-name>` is a **placeholder**. Replace it, including the `< >` brackets, with your own value. For example, `cd <repo-name>` becomes `cd ds-visualisation`.

### 1. Create the Repository from the Template

Click **Use this template** on GitHub.

When creating the repository:

- Set yourself as the **Owner**
- Choose a repository name
- Disable **Include all branches**
- Click **Create repository**

> [!IMPORTANT]
> If you are working in pairs or groups, only **one person** should complete this step.

---

### 2. Add Collaborators (Pairs/Groups Only)

If working with teammates:

1. Open the repository on GitHub
2. Go to **Settings → Collaborators**
3. Add your teammates as collaborators
4. Share the repository link with your team

Teammates should accept the invitation before continuing.

---

### 3. Clone the Repository

Copy the SSH URL from the **Code** button on GitHub, then run:

```bash
git clone <copied-ssh-url>
```

The copied SSH URL will look like `git@github.com:<your-username>/<repo-name>.git`.

---

### 4. Move into the Project Folder and Install Dependencies

This installs all dependencies and creates a virtual environment in `.venv/`.

```bash
cd <repo-name>
uv sync
```

---

### 5. Set up your database credentials

The [**Fetching the Data**](3_Fetching_the_data.ipynb) notebook connects to a PostgreSQL database. Copy the example file, then fill in the values your coaches give you:

```bash
cp .env.example .env
```

Open `.env` and replace the placeholders (`USER_DB`, `PASSWORD`, and the host) with your own credentials.

> [!CAUTION]
> Never commit `.env`: it holds secrets and is already git-ignored. Only `.env.example`, with placeholders, belongs in the repository.

---

### 6. Open the Notebooks

> [!NOTE]
> Open VS Code from the project root so it automatically detects the environment created by `uv sync`.

Launch VS Code in the project root:

```bash
code .
```

Then open a notebook and select the Python environment created by `uv sync` as the kernel.

## References & Further Reading

- [**Python Plotting With Matplotlib (Real Python)**](https://realpython.com/python-matplotlib-guide/): A thorough walkthrough of Matplotlib's figure and axes model.
- [**Seaborn user guide and tutorial**](https://seaborn.pydata.org/tutorial.html): The official Seaborn tutorial covering statistical plots and styling.
- [**Plotly Python Graphing Library**](https://plotly.com/python/): Official documentation for Plotly's interactive charts.
- [**Jupyter Notebooks in VS Code**](https://code.visualstudio.com/docs/datascience/jupyter-notebooks): How to run and debug notebooks inside VS Code.
- [**From Data to Viz**](https://www.data-to-viz.com/): A decision tree for picking the right chart type, with common caveats to avoid.
- [**The Python Graph Gallery**](https://python-graph-gallery.com/): Hundreds of reproducible chart examples in Matplotlib, Seaborn, and Plotly.
