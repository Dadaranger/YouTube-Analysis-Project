# YouTube Trending Video Analysis

> **Navigation Guide**: This project has been consolidated into two comprehensive documentation files. Choose the one that best fits your needs:

---

## [README_PROJECT.md](./README_PROJECT.md) - Technical Documentation
**For developers, researchers, and those interested in methodology**

Contains:
- **Complete Analysis Pipeline**: Step-by-step breakdown of all analysis stages
- **Recent Updates & Fixes**: Latest improvements and bug fixes
- **Statistical Methodology**: Why we use `median log1p(views)` (with proof!)
- **Data Processing**: ETL pipeline, feature engineering, validation steps
- **Code Implementation**: Python patterns, reproducibility guide
- **Technical Details**: Time zone handling, outlier treatment, sample size considerations

 **Read this if you want to:**
- Understand or reproduce the analysis
- Learn the technical implementation
- See the statistical reasoning behind our choices
- Review code patterns and data processing steps

---

## [README_FINDINGS.md](./README_FINDINGS.md) - Key Insights & Recommendations
**For content creators, marketers, and those seeking actionable insights**

Contains:
- **Executive Summary**: Top 3 game-changing discoveries
- **Best Posting Times**: When to publish for maximum views (Saturday 3PM ET!)
- **Text Strategy**: What to write (descriptions matter 7x more than titles!)
- **Content Optimization**: DOs and DON'Ts backed by data
- **Punctuation Guide**: Emojis hurt performance (-26%), numbers help (+8%)
- **Format-Specific Tips**: Different strategies for Shorts, Music, etc.
- **Statistical Confidence**: How reliable are these recommendations?

 **Read this if you want to:**
- Optimize your YouTube content strategy
- Learn what drives video performance
- Get actionable recommendations backed by 13K+ videos
- Understand what works (and what doesn't) on trending

---

## Quick Project Overview

**Dataset**: 13,017 YouTube trending videos (July-September 2024, US)

**Analysis Components**:
1. **Temporal Analysis**: Best days/hours for posting (Step 2)
2. **Text Feature Analysis**: Title/description optimization (Step 3)
3. **Sentiment Analysis**: How tone affects performance (Step 3.5)

**Main Notebook**: `research.ipynb` - Complete analysis pipeline with visualizations

**Key Discovery**: Our log transformation validation proved that **raw view counts are 3.56x skewed** toward outliers. Using `median log1p(views)` provides recommendations that work for **typical creators**, not just viral anomalies.

---

## Quick Start

1. **For Content Creators**: Jump straight to [README_FINDINGS.md](./README_FINDINGS.md) → "Actionable Recommendations"

2. **For Technical Review**: Start with [README_PROJECT.md](./README_PROJECT.md) → "Analysis Methodology"

3. **For Reproducibility**: See [README_PROJECT.md](./README_PROJECT.md) → "Reproducibility Guide"

---

## Project Structure

```
YouTube Analysis Project/
├── README.md (you are here - navigation index)
├── README_PROJECT.md (technical documentation)
├── README_FINDINGS.md (insights & recommendations)
├── research.ipynb (main analysis notebook)
├── Data/ (raw CSV files by date)
├── out_step2_temporal/ (temporal analysis results)
├── out_step3_5/ (text & sentiment analysis results)
└── requirements.txt (Python dependencies)
```

---

## Contact & Questions

**Having trouble deciding which README to read?**

-  **Technical questions** (methodology, code, statistics) → [README_PROJECT.md](./README_PROJECT.md)
-  **Strategy questions** (what works, best practices) → [README_FINDINGS.md](./README_FINDINGS.md)
-  **Both?** Start with FINDINGS for insights, then dive into PROJECT for details!

---

**Last Updated**: October 2025  
**Dataset Period**: July-September 2024  
**Videos Analyzed**: 13,017 trending videos
