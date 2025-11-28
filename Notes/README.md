
## Spotify What Is Known (2025 Evidence Snapshot)

This guide builds on a converging body of public research, platform documentation, and industry reverse-engineering about how Spotify works and how playlist fraud shows up in the data. At a high level:

### 1. How Spotify’s Recommendation System Behaves

* Spotify’s core stack can be described as **BaRT-style bandits over a hybrid recommender**:

  * **Collaborative filtering** (user–user, item–item, matrix factorization) maps users and tracks into a shared embedding space.
  * **NLP models** embed text and culture: playlist titles, bios, reviews, social chatter, lyrics, and editorial copy into the same space as tracks.
  * **Audio analysis** (Echo Nest lineage) embeds raw audio via features like tempo, key, loudness, MFCCs, energy, valence, etc.
* A contextual bandit / learning-to-rank layer decides what to play next given:

  * User state (taste profile, session behavior, time of day)
  * Track and playlist features
  * Multi-objective goals (keep sessions going, introduce new music, respect business constraints like Discovery Mode).

### 2. Engagement Metrics That Actually Matter

Across Spotify for Artists, label dashboards, and third-party analytics, a consistent picture emerges:

* **Repeat Listen Rate (streams per listener)**
  ~2.0 is “good,” 3.0+ is “very strong” and correlates with algorithmic lift.
* **Save Rate**
  20%+ of listeners saving a track is considered excellent; saves are one of the strongest predictors of long-term streams.
* **Completion vs Skip Rate**
  The first 30 seconds are critical; high early skip rates are heavily penalized, while high completion rates are rewarded.
* **Playlist Adds**
  When listeners add a track to their own playlists, it acts as a high-value endorsement, helping the song jump networks.
* **Programmed vs Active Audience**
  Sustainable careers need both:

  * Programmed: Discover Weekly, Radio, Autoplay, algorithmic playlists.
  * Active: direct searches, profile plays, followers, library plays.

Quality playlist placements matter because they **amplify these metrics**, which in turn drive inclusion in Discover Weekly, Release Radar, Radio, and “Fans also like.”

### 3. What a “Good” vs “Bad” Playlist Looks Like

Empirically, **good playlists** share these traits:

* Steady follower growth, not overnight spikes.
* Reasonable relationship between:

  * Playlist followers and curator followers
  * Streams, saves, and follower growth for artists on the list.
* Coherent curation (clear mood/genre/use-case, decent track count, regular updates).
* Healthy engagement for your tracks: decent save rate, repeat listens, completion, and playlist adds.

**Bad or botted playlists** typically show:

* Sudden follower spikes or purges with no visible marketing reason.
* Extreme listener-to-stream ratios (e.g., 100:1+ or perfectly 1:1) and zero saves.
* Streams clustered in odd geographies with no corresponding social or cross-platform signals.
* Weak curation signals: tiny or enormous track counts, no consistent theme, low diversity, or one artist dominating the list.
* Curators with almost no followers, no bio, no socials, or explicit “guaranteed streams/placement” language.

### 4. Proven Detection & Verification Signals

Modern playlist-verification tools and fraud-detection research rely on:

* **Time-series growth analysis**: velocity, volatility, spikes, and purges in follower counts.
* **Engagement vs scale checks**: streams, saves, completion, and listener counts benchmarked against follower size.
* **Structural features**: track count, artist diversity, genre/mood cohesion, update frequency.
* **Curator authentication**: profile completeness, cross-platform consistency, social presence, and history of credible placements.
* **ML and anomaly detection**: tree-based models and isolation/one-class methods over these features to flag high-risk playlists.

This “what is known” layer is the empirical foundation for the system described in the rest of the guide: everything from schema design and feature engineering to ML models and business rules is built to operationalize these observed patterns into a rigorous, reproducible verification score.
