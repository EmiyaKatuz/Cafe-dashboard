# Project Approach: Cafe Feedback Dashboard

## 1. Introduction

This document outlines the analytical and technical approach taken to build the Cafe Feedback Dashboard. The goal of this project was to transform raw, messy feedback data into an interactive tool that allows store managers and stakeholders to monitor customer satisfaction, sales performance, and qualitative feedback.

## 2. Data Analysis Perspective

### 2.1. Data Cleaning & Preprocessing

The raw data presented several quality issues that required a robust cleaning pipeline before any analysis could be trusted. The exploration phase (documented in `cafe_feedback_exploration.ipynb`) revealed:

* **Inconsistent Location Names:** Entries like "Albany" and "Albany " (with trailing spaces) were treated as separate entities.
  * *Approach:* I implemented string normalization (trimming and regex whitespace replacement) to merge these duplicates.
* **Dirty Transaction Values:** The `Transaction Value` column contained non-numeric artifacts, including timestamp-like strings (e.g., `20/10/2020 8:24:00 AM`) and extreme outliers.
  * *Approach:* A custom parsing function was written to extract currency values using regex. I also applied a logical range filter ($0 < x \le 500$) to exclude data entry errors and timestamp artifacts.
* **Date Parsing:** Timestamps were in a day-first format.
  * *Approach:* Explicitly parsed using `pd.to_datetime(..., dayfirst=True)` to ensure accurate daily trend analysis.

### 2.2. Key Metrics & KPIs

To provide a high-level overview, I selected four core metrics:

1. **Volume:** Total number of feedback records (indicates data significance).
2. **Satisfaction:** Average Rating (1-5 scale).
3. **Revenue:** Average Transaction Value (ATV).
4. **Typical Spend:** Median Transaction Value (robust against outliers).

### 2.3. Exploratory Analysis Logic

* **Rating vs. Spend:** I investigated whether higher ratings correlate with higher spending. The "Average transaction value by rating" chart was designed to test the hypothesis that happier customers spend more.
* **Text Analysis:** Instead of heavy NLP models, I opted for a frequency-based approach.
  * *Stop Words Definition:* Currently, a **manual list** of stop words (e.g., "the", "and", "is") is defined in the code. This gives us precise control to ensure domain-specific terms aren't accidentally filtered out.
  * *Automatic Alternatives:* We can also use **automatic stop word lists** from libraries like `wordcloud` (which has a built-in `STOPWORDS` set), `nltk`, or `spaCy`. These are more comprehensive but might require customization to avoid removing words that are meaningful in a cafe context.

### 2.4. Visualization Strategy

Every chart in the dashboard serves a specific business question:

* **Rating Distribution (Bar Chart):**
  * *Why:* Averages hide extremes. A 4.2 average could mean everyone gave a 4, or half gave 5 and half gave 3. This chart reveals the *consistency* of service.
* **Average Transaction by Rating (Bar Chart):**
  * *Why:* This directly links CX (Customer Experience) to ROI. If 5-star ratings correlate with higher basket sizes, it justifies investment in service training.
* **Daily Trends (Line Chart):**
  * *Why:* Aggregated data hides momentum. A store might have a great average but a crashing trend over the last week. This view helps managers spot "bad days" (e.g., a machine breakdown or understaffing event).
* **Top Words (Bar Chart) & Word Cloud:**
  * *Why:* Managers don't have time to read 2,000 comments. These visuals bubble up the "loudest" topics immediately, distinguishing between product issues ("coffee") vs. service issues ("wait", "rude").

### 2.5. Automated Narrative Generation

A unique feature of this dashboard is the "AI-style summary."

* *Logic:* Instead of using a black-box LLM, I implemented a deterministic rule-based system (`build_narrative`).
* *Why:* This ensures the summary is always factually grounded in the currently filtered data. It dynamically inserts calculated stats (e.g., top location, busy days) into a pre-structured narrative template, providing "human-readable" insights instantly.

## 3. Technical Perspective

### 3.1. Technology Stack

* **Python:** The core language, chosen for its dominance in data science.
* **Streamlit:** Selected as the application framework.
  * *Reasoning:* Streamlit allows for rapid prototyping and turns data scripts into shareable web apps with minimal frontend code. It handles the UI widgets (sliders, dropdowns) automatically.
* **Pandas:** Used for all data manipulation, filtering, and aggregation.
* **Plotly Express:** Used for visualization.
  * *Reasoning:* Unlike static Matplotlib charts (used in the exploration notebook), Plotly charts are interactive. Users can hover over bars to see exact values, zoom in on trends, and toggle series, which is essential for a dashboard experience.

### 3.2. Architecture & Performance

* **Caching:** The `@st.cache_data` decorator is applied to the `load_data` function.
  * *Benefit:* Data cleaning is an expensive operation. Caching ensures that the raw CSV is parsed and cleaned only once. Subsequent interactions (changing filters) use the cached dataframe, making the app snappy.
* **Component-Based Design:** The code is structured into functional blocks (Loading, Layout, Sidebar, KPIs, Charts). This makes the codebase maintainable and easy to extend.

### 3.3. User Experience (UX) Design

* **Sidebar Filters:** Controls are placed in the sidebar to keep the main view focused on insights.
* **Drill-Down Capability:** Users can filter by Location, Rating, and Date. The dashboard reacts instantly, allowing a manager to answer specific questions like *"Why was the rating low in Albany last week?"*
* **Downloadable Data:** A "Download cleaned data" button is provided, acknowledging that some users may want to perform their own analysis in Excel.

## 4. Thought Process Summary

The development followed a "Data-First" methodology:

1. **Inspect:** I started by profiling the raw data in a notebook to understand its quirks.
2. **Clean:** I wrote targeted cleaning rules to address specific issues found during inspection.
3. **Visualize:** I built the dashboard to surface the most valuable signals (Ratings, Spend, Trends).
4. **Narrate:** I added the text summary to bridge the gap between raw numbers and actionable business advice.

This approach ensures that the final tool is not just a display of data, but a reliable decision-support system.
