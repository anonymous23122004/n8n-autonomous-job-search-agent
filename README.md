# Autonomous Job Search Agent (n8n & Generative AI)

An enterprise-grade autonomous job hunting pipeline built with **n8n**, **Google Gemini**, **Apify**, **Supabase**, and **LaTeX** compilation engines. The system automatically scrapes job listings, evaluates candidate-job fit, dynamically refructures and tailors resumes based on target profiles, compiles PDFs, and drafts recruiter outreach messages.

---

## 🚀 Key Features

*   **LinkedIn Scraping (Apify)**: Automated extraction of the latest job postings based on customized search queries.
*   **100% ATS-Compliant Resume Tailoring (Gemini)**: Leverages the industry-standard **Jake's Resume** LaTeX template. Generates single-column, machine-readable PDFs with native Unicode support (`\pdfgentounicode=1`) and zero parsing corruption (no Alt-Text symbol leaks).
*   **Duplicate Filtering**: Integrates with a Supabase database to avoid processing duplicate job listings.
*   **Closed-Loop Learning Database**: Tracks resume versions, application statuses, and recruiter responses.
*   **Gmail Integration**: Sends automated daily reports summarizing processed, applied, and rejected job listings.

---

## 🛠️ System Architecture Upgrades (Fresher Strategy)

This repository includes a blueprint for **8 additive upgrades** designed to maximize interview conversion rates for B.Tech CS (AI/ML) fresher profiles targeting non-coding AI roles:

1.  **Job Intelligence Layer**: Identifies coding-heavy keywords and senior roles, automatically flagging roles for rejection before running expensive LLM runs.
2.  **Career Relevance Scoring**: Compares candidates against the JD, assigning Match, Fresher, and Coding Scores.
3.  **Multi-Resume Selector**: Automatically selects between 4 specialized base resume families (**Prompt Engineering**, **AI Evaluation**, **AI Operations**, **Customer Success**) using strict candidate integrity rules (no information fabrication).
4.  **Company Context Summary**: Analyzes job text to construct a zero-cost summary of target business lines to align the resume summary and bullet points.
5.  **Conditional Recruiter Outreach**: Dynamically drafts LinkedIn (InMail), email follow-ups, and cold emails only for high-fit roles (Match Score $\ge$ 90) in target categories.
6.  **Skill Gap Analyzer**: Compares rejected JDs against candidate resumes, extracting missing skills and learning priorities into a `skill_gap_analysis` Supabase table.
7.  **Platform Priority Weighting**: Boosts weighting for job listings found on specialized AI evaluation marketplaces (e.g. Outlier, Alignerr, Scale AI, DataAnnotation) over generic boards.
8.  **Weekly Strategy Email**: Sends a macro-level performance summary (top missing skills, platform conversion rates) to continuously tune the job search strategy.

---

## ⚙️ Setup & Deployment Guide

### 1. Prerequisite Accounts
*   **n8n**: A local or cloud-hosted instance.
*   **Google AI Studio**: Free API key for Gemini 2.5 Flash.
*   **Apify**: Free account for LinkedIn scraping credits.
*   **Supabase**: Free Postgres database instance.
*   **Google Drive & Gmail**: For document retrieval, storage, and emails.

### 2. Supabase Table Setup
Execute the following schema migrations in your Supabase SQL Editor:
```sql
-- Main processed jobs tracking
CREATE TABLE IF NOT EXISTS jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_url TEXT UNIQUE NOT NULL,
    job_title VARCHAR(255) NOT NULL,
    company VARCHAR(255) NOT NULL,
    processed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tailored resume tracking
CREATE TABLE IF NOT EXISTS resume_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_url TEXT NOT NULL,
    resume_family VARCHAR(100) NOT NULL,
    tailored_resume_text TEXT NOT NULL,
    match_score INT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Recruiter outreach tracking
CREATE TABLE IF NOT EXISTS recruiter_outreach (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_url TEXT NOT NULL,
    linkedin_message TEXT,
    follow_up_email TEXT,
    cold_email TEXT,
    outreach_status VARCHAR(50) DEFAULT 'Generated',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Skill gap logs (Fail branch of Relevance Gate)
CREATE TABLE IF NOT EXISTS skill_gap_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_url TEXT UNIQUE NOT NULL,
    company VARCHAR(255) NOT NULL,
    role VARCHAR(255) NOT NULL,
    missing_skills TEXT[] NOT NULL,
    recommended_keywords TEXT[] NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 3. Import Workflow
1.  Download the `workflow.json` file in this repository.
2.  Open your n8n dashboard and click **Import from File**.
3.  Locate the `"Workflow Configuration1"` node.
4.  Configure the assignments:
    *   `resumeFileId`: The file ID of your master resume in Google Docs.
    *   `userEmail`: Your Gmail address.
    *   `apifyToken`: Your Apify personal API token.
    *   `supabaseUrl` / `supabaseKey`: Your Supabase project URL and anonymous key.
5.  Link your Google Drive, Supabase, and Gmail credentials.
6.  Activate the workflow!
