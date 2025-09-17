## Project Steps Overview (1–3)
### Step 1 — Dataset Preparation & Cleaning

We began by combining multiple monthly trending datasets into a single dataset.
The logic was:

Ensure we had a complete and consistent dataset covering the full time period.

Identify missing values, duplicates, and inconsistent formats.

Drop unreliable fields (e.g., creatorOnRise, which was missing in most rows).

Convert invalid durations (≤0 or -1) into proper missing values.
This step ensures we are working with clean, reliable data before making any conclusions.

### Step 2 — Temporal Analysis (When to Post)

Next, we analyzed posting times to understand how timing affects video performance.
The logic was:

Convert video timestamps into local time (ET) for consistency.

Create hour/day features to compare publishing windows.

Use medians and quantiles of views (instead of averages) to avoid distortion from viral outliers.

Break results into meaningful groups (all videos, long-form vs Shorts, verified vs unverified).
This step shows us how timing interacts with video format and authority level.

### Step 3 — Title & Description Analysis (What to Say)

Finally, we analyzed the text features of titles and descriptions.
The logic was:

#### Step 3.1: Measure lengths (characters/words) → to see if there is a sweet spot for readability.

#### Step 3.2: Track punctuation and symbols (e.g., numbers, “?”, “!”, emojis) → to see how style choices matter.

#### Step 3.3: Identify most common keywords in titles/descriptions → to understand language trends.

#### Step 3.4: Approximate sentiment/tone of titles and descriptions (positive, neutral, negative) → to see how tone might influence outcomes.
Each substep breaks the problem into one type of textual signal at a time, so we can later combine them into a predictive model.