# **Humanitarians AI — Using AI to Recommend Legit Music Playlists**

### *A Work-in-Progress YouTube Course Built by Volunteers Learning-by-Teaching*

**A Program of Humanitarians AI, a 501(c)(3) nonprofit dedicated to ethical AI for social good.**
[https://www.humanitarians.ai/](https://www.humanitarians.ai/)

---

# **Course Overview**

**Humanitarians AI — Using AI to Recommend Legit Music Playlists** is a community-driven, open-access educational project where volunteers—often recent graduates, students, and early-career technologists—**learn by teaching**.

This course teaches students to build an **AI-powered playlist verification system** that detects bot-farmed playlists and recommends legitimate promotional opportunities for independent artists. Students will learn:

* **Machine Learning fundamentals** through real-world music industry fraud detection
* **Time-series analysis** to identify suspicious growth patterns
* **Audio feature analysis** using Spotify's API
* **Ethical AI practices** in the music promotion ecosystem
* **Full-stack development** from data collection to deployment

This course is a **work in progress**, evolving continuously through:

* Iterative outlines
* Community-created lecture content
* GitHub-hosted hands-on labs
* Real-time improvements using the **Humanitarians Courses GPT**
  [https://chatgpt.com/g/g-6920a81baeb8819185cc8100ce963e95-humanitarians-courses](https://chatgpt.com/g/g-6920a81baeb8819185cc8100ce963e95-humanitarians-courses)

Each course aims to provide **ethical, accessible, inclusive AI education**, aligned with the mission of Humanitarians AI to *use AI for the greater good*—especially for underserved communities in education, health, nonprofits, and the arts.

---

# **Course Structure & Pedagogy**

Each Humanitarians AI YouTube course follows a **"Learn → Build → Teach"** cycle:

1. **Start with an Outline**
   Volunteers begin with a structured outline (like this one) and refine it collaboratively.

2. **Develop Hands-On Labs First**
   Before any lecture videos are recorded, the course repository includes:

   * Jupyter notebooks
   * Code demos
   * Data labs
   * Mini-projects
   * Applied exercises

3. **Use the Humanitarians Courses GPT**
   Volunteers generate:

   * Lecture scripts
   * Slides
   * Lab guides
   * Assignments
   * Explanations, demos, and analogies

4. **Produce YouTube Lectures**
   Once labs & notebooks are complete, volunteers record and upload:

   * Concept walkthroughs
   * Coding tutorials
   * Mini-lectures
   * Guest interviews

5. **Continuous Updating**
   New fellows and contributors continue improving the course, adding new labs, example datasets, and recorded demos.

---

# **Part I — Foundations: Understanding the Music Streaming Ecosystem**

---

## **Module 1 — The Music Promotion Crisis: Why This Matters**

Overview of the independent music industry and the playlist promotion fraud problem.

**Topics include:**

* The role of streaming playlists in music discovery (2025 landscape)
* How Spotify's algorithm actually works (BaRT, collaborative filtering, engagement metrics)
* The economics of streaming: legitimate promotion vs. bot farms
* Real consequences: artists losing catalogs, distributors facing penalties
* **Ethical considerations**: Why building transparent verification tools matters
* **Social impact**: Helping independent artists compete fairly

**Lab 1.1:** Explore Spotify for Artists data - analyze real engagement metrics

---

## **Module 2 — How Spotify's Algorithm Works (and Why Bots Break It)**

Deep dive into recommendation systems and engagement signals.

**Includes:**

* Collaborative filtering and matrix factorization
* Audio feature analysis (Echo Nest legacy)
* Natural language processing for metadata
* Critical engagement metrics:
  - Save rate (20%+ = algorithmic boost)
  - Stream-to-listener ratio
  - Skip rate and completion rate
  - Playlist adds
* How bot streams corrupt these signals
* The multiplication effect: playlist → algorithm → discovery

**Lab 2.1:** Reverse-engineer Spotify's audio features using Librosa
**Lab 2.2:** Visualize the algorithm's decision boundary with real playlist data

---

## **Module 3 — Tools & Data Sources for Playlist Analysis**

Hands-on introduction to the technical stack.

**Covers:**

* Spotify Web API setup and authentication
* **Spotipy** library (Python wrapper)
* Available vs. unavailable data (API limitations)
* Alternative data sources:
  - Chartmetric API (historical follower data)
  - Your own artist analytics
  - Web scraping techniques (ethical considerations)
* Database design: PostgreSQL/SQLite for time-series data
* Data pipeline architecture

**Lab 3.1:** Build your first Spotify API scraper
**Lab 3.2:** Design a database schema for playlist monitoring
**Lab 3.3:** Extract audio features for 1000 tracks

---

# **Part II — Building the Detection System**

---

## **Module 4 — Statistical Red Flags: The Math of Bot Detection**

Core detection algorithms and statistical methods.

**Includes:**

* **Timestamp analysis**: Detecting batch uploads
  - Code: `added_at` clustering with pandas
* **Popularity distribution**: Ghost playlist detection
  - Statistical test: Is this distribution normal?
* **Audio coherence**: Sonic chaos detection
  - Feature: Energy/tempo standard deviation
  - Sweet spot: 0.1-0.25 STD
* **Follower growth patterns**: Staircase detection
  - Time-series anomaly detection
* **Curator credibility scoring**: Multi-factor authentication

**Lab 4.1:** Implement batch upload detector (10+ tracks/minute = flag)
**Lab 4.2:** Build ghost playlist detector (10K+ followers, <10 avg popularity)
**Lab 4.3:** Calculate audio feature standard deviation
**Lab 4.4:** Detect follower growth spikes with moving averages

---

## **Module 5 — The Monitor System: Time-Series Tracking**

Building the persistent tracking infrastructure.

**In-depth application:**

* Why snapshots aren't enough: The monitoring requirement
* Cron jobs and scheduled data collection
* SQLite/PostgreSQL time-series tables
* Detecting patterns over days/weeks:
  - +1000 followers in 1 hour → flatline = bot purchase
  - Gradual organic growth vs. staircase patterns
* Data visualization: Follower graphs, track turnover charts
* Alert systems: Email/Slack notifications for suspicious activity

**Lab 5.1:** Build a daily cron job to track 100 playlists
**Lab 5.2:** Create follower growth visualization dashboard
**Lab 5.3:** Implement alerting system for anomaly detection

---

## **Module 6 — Machine Learning Models for Classification**

From heuristics to trained models.

**Students learn how to:**

* Feature engineering for playlist data:
  - 20+ engineered features from raw data
* Supervised learning: Labeling known bot vs. legitimate playlists
* Training classifiers:
  - Logistic regression (baseline)
  - Random Forest (production model)
  - XGBoost (advanced)
* Unsupervised anomaly detection:
  - Isolation Forest
  - One-Class SVM
* Model evaluation: Precision, recall, F1-score
* Handling imbalanced datasets
* Model interpretation: SHAP values for explainability

**Lab 6.1:** Create labeled training dataset (100 playlists)
**Lab 6.2:** Train Random Forest classifier with scikit-learn
**Lab 6.3:** Implement anomaly detector for zero-day bot detection
**Lab 6.4:** Build SHAP explainability dashboard

---

# **Part III — Advanced Topics, Production & Impact**

---

## **Module 7 — Advanced Detection & Genre Matching**

Deeper technical challenges.

**Topics:**

* Natural language processing for curator verification:
  - Analyzing playlist descriptions for spam phrases
  - Cross-platform identity verification
* Audio embeddings for genre matching:
  - Using pre-trained models (MusicNN, VGGish)
  - Cosine similarity for track-playlist compatibility
* Handling edge cases:
  - New playlists with no history
  - Legitimate viral growth vs. bot spikes
  - Editorial playlists (always legitimate)
* Active learning: Improving model with user feedback

**Lab 7.1:** Build NLP classifier for spam descriptions
**Lab 7.2:** Implement audio embedding similarity search
**Lab 7.3:** Create genre-matching recommendation engine

---

## **Module 8 — Social Impact Lab: Ethical Music Promotion**

Direct alignment with Humanitarians AI's mission.

**Students explore:**

* How bot farms harm independent artists:
  - Diluted royalty pools
  - Algorithmic punishment for fake engagement
  - Lost career opportunities
* Economic impact on underrepresented musicians:
  - Artists of color
  - International artists
  - Protest and social justice music (like "Mayfield King")
  - Spiritual and faith-based genres
* Building transparent alternatives to scam services
* **Case study**: Using the system for Humanitarians AI artists
* Open-source release: Making tools available to all independent musicians
* **Policy recommendations**: Industry-wide fraud prevention

**Lab 8.1:** Analyze economic impact of bot farms on indie artist revenues
**Lab 8.2:** Create "Ethical Playlist Promotion" best practices guide
**Lab 8.3:** Test system on real Mayfield King tracks

---

## **Module 9 — Capstone: Deploy Your Playlist Verification Service**

Each learner creates a working system:

**Project options:**

1. **Full-stack web app** (Flask/FastAPI + React)
   - User submits playlist URL → receives risk score + report
   - Database of verified "safe" playlists
   - Artist dashboard for tracking their placements

2. **Discord/Telegram bot**
   - Musicians paste Spotify links in chat
   - Bot returns instant verification analysis
   - Community moderation features

3. **API service for distributors**
   - B2B tool for labels/distributors
   - Batch playlist verification
   - Integration with existing systems

4. **Research paper**
   - Academic publication on detection methods
   - Dataset release for research community
   - Comparative analysis of existing tools

5. **Open dataset**
   - 10,000+ labeled playlists (bot vs. legitimate)
   - Released on Kaggle/HuggingFace
   - Documentation and benchmarks

**Required components:**

* Working code deployed somewhere (GitHub, Heroku, Vercel)
* Documentation and usage guide
* Alignment with **AI for Good** principles
* Contribution back to course repository

**Lab 9.1:** Choose your capstone project
**Lab 9.2:** Build MVP in 2 weeks
**Lab 9.3:** User testing with real musicians
**Lab 9.4:** Open-source release and documentation

---

# **Course Objectives**

By the end of this course, volunteers will be able to:

### **Understanding**

* Explain how Spotify's recommendation algorithm works
* Understand the mechanics of streaming fraud
* Articulate ethical implications of AI in music promotion
* Identify proxy metrics for detecting bot activity

### **Technical Skills**

* Use Spotify API and Spotipy library
* Build time-series data pipelines with PostgreSQL
* Implement statistical anomaly detection
* Train and deploy machine learning classifiers
* Create audio feature analysis systems
* Design full-stack web applications
* Build REST APIs with Flask/FastAPI

### **Teaching Skills**

* Create tutorials on ML concepts
* Record coding walkthroughs
* Explain complex algorithms simply
* Use Humanitarians Courses GPT to support content creation
* Document technical projects clearly

### **Social Impact**

* Identify how AI can protect independent artists
* Build transparent, ethical alternatives to scam services
* Contribute to open-source music tech
* Support underrepresented musicians

### **Career Skills**

* Portfolio project for music tech/ML roles
* Experience with real-world fraud detection
* Understanding of streaming industry
* Open-source contributions

---

# **Course Materials**

### **Primary Resources**

* GitHub course repository (with 100+ labeled playlists dataset)
* Jupyter notebooks & hands-on labs
* Humanitarians Courses GPT assistant
* Sample Spotify API credentials (rate-limited)
* Pre-scraped playlist data (to avoid API limits)

### **Additional Materials**

* Academic papers on recommendation systems
* Spotify Research blog posts
* Music industry fraud reports
* Open-access datasets (Million Song Dataset, etc.)
* Community-created tutorials
* YouTube lectures (as they are produced)
* **Guest lectures from**:
  - Music distribution platforms
  - Spotify for Artists experts
  - Independent musician success stories
  - Nik Bear Brown (course creator) on Mayfield King project

---

# **Course Highlights**

* **Real-world impact**: Tools that actually help musicians
* Built by volunteers learning by teaching
* **Portfolio-worthy capstone**: Deployable applications
* Integrated course chatbot + GEM agent
* Strong alignment with **AI for Good**
* **Music industry relevance**: Skills applicable to streaming/tech jobs
* Designed for beginners, recent grads, and new fellows
* Evergreen updating through Humanitarians AI
* **Cross-promotional**: Features Mayfield King music in examples
* **Community building**: Discord for musicians and developers

---

# **Instructors, Contributors & Approvals**

* **Created by**: Nik Bear Brown (Associate Teaching Professor, Northeastern University)
  - Founder: Humanitarians AI & Musinique LLC
  - Artist: Mayfield King (protest & spiritual music)
* **Course model**: Learn by teaching - volunteers become instructors
* No instructor approval required for contributions
* Open to contributors at any skill level
* All contributions reviewed through GitHub pull requests
* **Featured case studies**: Real Mayfield King tracks and playlists

---

# **📁 Repository Structure**

This is the **standard Humanitarians AI course repository layout**.

---

## **Core Course Materials**

* **`Outline/`** — Initial outline (THIS DOCUMENT)
* **`Labs/`** — Jupyter notebooks, coding exercises, datasets
* **`Lectures/`** — Slides, scripts, recorded videos (added later)
* **`Assignments/`** — Problem sets & mini-projects
* **`Syllabus/`** — Syllabus + weekly plan

---

## **Topic Modules (Auto-Generated Over Time)**

* **`Module_01_Music_Industry_Crisis/`**
* **`Module_02_Spotify_Algorithm/`**
* **`Module_03_Tools_Data_Sources/`**
* **`Module_04_Statistical_Detection/`**
* **`Module_05_Time_Series_Monitoring/`**
* **`Module_06_ML_Classification/`**
* **`Module_07_Advanced_Detection/`**
* **`Module_08_Social_Impact/`**
* **`Module_09_Capstone_Deployment/`**

---

## **Learning Resources**

* **`Datasets/`** 
  - `labeled_playlists.csv` (100+ bot/legit classifications)
  - `mayfield_king_analytics.csv` (real artist data)
  - `spotify_audio_features.parquet` (10K tracks)
* **`Cheatsheets/`** 
  - Spotify API quick reference
  - SQL queries for time-series
  - ML model evaluation metrics
* **`Reading/`** 
  - Research papers on recommendation systems
  - Spotify Research blog archives
  - Music industry fraud reports
* **`Notebooks/`** 
  - All labs sorted by module
  - Complete working examples
* **`MD/`** 
  - Markdown notes, FAQs, how-tos
  - Spotify API gotchas
  - Deployment guides

---

## **Tooling**

* **`GEM/`** — The course's Generative Education Model agent
* **`GPT_Assistant/`** — Course ChatGPT assistant configuration
  (Both ingest the GitHub repo as their knowledge base)
* **`API_Keys/`** — Instructions for getting Spotify credentials
* **`Docker/`** — Containerized development environment

---

## **Setup Instructions**

```bash
git clone https://github.com/HumanitariansAI/AI-Playlist-Verification.git
cd AI-Playlist-Verification
git checkout organized_branch

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up Spotify API credentials
cp .env.example .env
# Edit .env with your Spotify Client ID & Secret

# Launch Jupyter
jupyter lab
```

---

## **Sample Projects Included**

* **`example_bot_detector.py`** - Complete working detector
* **`mayfield_king_analyzer.py`** - Analyze playlists for protest music
* **`dashboard_app/`** - Flask web app for playlist verification
* **`discord_bot/`** - Ready-to-deploy Discord bot
* **`datasets/verified_playlists/`** - Curated safe playlists for indie artists

---

# **License**

All content is open educational material for humanitarian use.
Code licensed under MIT.
Course content licensed under CC BY-SA 4.0.

---

# **Join the Mission**

Humanitarians AI is a nonprofit dedicated to **ethical, accessible AI** for global good.

Support us or get involved:
[https://www.humanitarians.ai/](https://www.humanitarians.ai/)

**Support independent music:**
Listen to Mayfield King on Spotify: [Your Spotify Artist Link]
Explore protest & spiritual music: [Musinique Records Link]

---

# **Course Metadata**

* **Duration**: 9 modules (self-paced, ~40-60 hours total)
* **Prerequisites**: 
  - Basic Python programming
  - High school statistics
  - No ML experience required
* **Target audience**:
  - Musicians interested in tech
  - CS students wanting music industry experience
  - Data science beginners
  - Anyone passionate about ethical AI
* **Outcomes**: 
  - Deployed application
  - GitHub portfolio
  - Understanding of ML in production
  - Network in music tech community
