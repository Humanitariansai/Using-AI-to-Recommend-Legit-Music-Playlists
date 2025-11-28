
## 1. What “the Spotify algorithm” actually is

Spotify itself describes its system as a set of algorithms that select and order content across Home, Search, Radio, personalized playlists, and more.([Spotify][1]) Internally, this is roughly:

1. **Candidate generation**

   * “What could we show you right now?” (tens of thousands of tracks).
2. **Scoring / ranking**

   * “How likely are you to enjoy this if we show it?” (a relevance score per track).
3. **Re-ranking / constraints**

   * Diversity, novelty vs familiarity, promotion flags (e.g., Discovery Mode), policy constraints.
4. **Online learning**

   * Systems that adapt based on immediate feedback (skips, listens, etc.), often using contextual bandits / learning-to-rank approaches.([Chalmers Publication Library (CPL)][2])

Different surfaces (Discover Weekly, DJ, Made For You mixes, Radio, Autoplay, Search, etc.) share components but are tuned differently.([Spotify][3])

---

## 2. Data sources Spotify uses

From Spotify docs, engineering posts, and industry analyses, the main input signals are:([Spotify][1])

### 2.1 User behavior (implicit feedback)

These signals are heavily weighted because they reveal actual listening preferences:

* Plays, **skips (especially quick skips)**, and where in the song the skip happens
* Streams that pass the 30-second royalty threshold vs bounces in the first few seconds
* Completions and replays
* Adding a song to:

  * Library (Liked Songs)
  * Personal playlists
  * Queue
* Following the artist
* Sharing tracks or adding them to social stories
* Time-of-day, day-of-week, device type, and what else is played in the same session

Spotify summarizes this as your **“Taste Profile”**: a latent representation of your listening preferences used across Home, Discover Weekly, Blend, Wrapped, etc.([Spotify][4])

### 2.2 Audio content analysis

Spotify analyzes the audio itself using ML models, extracting features like tempo, key, mode, energy, acousticness, danceability, valence (mood), and more. These are exposed in the Web API and come from internal models trained on audio waveforms.([Beats To Rap On][5])

Modern analyses suggest deep neural networks (CNNs / transformer-style models) are used to embed tracks into high-dimensional “audio space,” so songs that *sound* similar are close together, even if they’re obscure or new.([JATIT][6])

### 2.3 Metadata and editorial signals

From distributor uploads and Spotify for Artists pitch forms:([music-tomorrow.com][7])

* Genre, sub-genre
* Mood and activity tags (chill, workout, sleep, focus, etc.)
* Instruments, languages, release type (single/EP/album)
* Country/region, release date
* Labels’ and editors’ annotations (e.g., “for Fresh Finds,” “for New Music Friday”)

These feed into candidate generation and also into **playlist fit**: e.g., does this track match the “coffeehouse indie” mood the playlist aims for?

### 2.4 Text / NLP signals

Spotify also scrapes and analyzes:([Beats To Rap On][5])

* Playlist titles and descriptions
* Editorial writeups and blog posts
* Music press, reviews, blogs, and maybe social chatter
* User-generated playlists (both public and private metadata patterns)

NLP models convert this text into embeddings that connect songs to **concepts** (“melancholic piano”, “driving techno,” “lofi study beats”).

---

## 3. Core algorithmic techniques

From academic papers, engineering posts, and reverse-engineering efforts, the backbone is:

### 3.1 Collaborative filtering & matrix factorization

* Spotify uses **implicit-feedback collaborative filtering**:

  * We have a giant user × track interaction matrix based on plays, skips, saves, etc.
  * Matrix factorization decomposes this into **user embeddings** and **track embeddings** in a shared space.([PyImageSearch][8])
* Recommendation = “find tracks whose vectors are close to this user’s vector.”

This powers things like **“Fans Also Like,” Discover Weekly, Daily Mixes**, etc., especially when there is lots of behavior data.

### 3.2 Content-based and hybrid models

For new or niche tracks where user interaction is sparse, Spotify leans more on:

* Audio embeddings (sound similarity)
* Metadata
* Textual context (NLP)

Then it blends this with collaborative signals as data accumulates — a **hybrid recommendation system**.([Beats To Rap On][5])

### 3.3 Contextual bandits & online learning to rank

Because Spotify must decide *in real time* what to play next (e.g., Radio / Autoplay), it uses **contextual bandit** and **online learning to rank** approaches:

* At each step, the system sees a **context** (user, device, time, prior songs, session mood).
* It chooses one or a slate of tracks (*actions*), gets a **reward** (play vs skip, dwell time, etc.).
* It balances:

  * **Exploitation**: play “safe” songs you almost surely like.
  * **Exploration**: try new tracks that might become future favorites.([Chalmers Publication Library (CPL)][2])

Academic and industrial work shows music platforms using contextual bandits to jointly optimize ranking and exploration over massive catalogs.([Amazon Science][9])

### 3.4 Multi-objective optimization

The system is not just optimizing user enjoyment. Objectives likely include:

* Short-term satisfaction (not skipping, keeping sessions going)
* Long-term retention (churn risk, subscription renewal)
* Content diversity and novelty
* Promotion and business goals (labels, Discovery Mode, new features)
* Legal and safety constraints (hate speech, explicit content in certain contexts, etc.)

Patents and industry analyses emphasize **multi-objective optimization** and real-time adaptation of recommendations.([PatentPC][10])

---

## 4. Where the algorithm shows up: main surfaces

Spotify’s personalization and design teams highlight a few key surfaces: Home, Search, and personalized playlists like Discover Weekly, Blend, and DJ.([Spotify][3])

### 4.1 Home

* A grid of rows (carousels), each powered by its own model:

  * “Jump back in”
  * “Made for [you]”
  * “Because you listened to X”
  * “New releases for you”
* Rows are **selected and ordered** based on your taste profile and recent activity.([Spotify][3])

### 4.2 Search & recommendations in search

* Autocomplete, “Top genres,” and “Browse all” are also personalized.
* Recommended content in Search is shaped by the same taste profile, but with more emphasis on **popular and trending** content for discovery.([Spotify][1])

### 4.3 Discover Weekly / Release Radar / Daily Mixes

* **Discover Weekly:** tries to give you ~30 tracks you haven’t heard before but are statistically likely to enjoy, heavily powered by collaborative filtering + exploration logic.([Spotify][11])
* **Release Radar:** new releases from artists you follow or might like.
* **Daily Mixes / Made For You:** more “comfort zone,” mixing familiar tracks with a smaller amount of exploration.

### 4.4 Radio / Autoplay

* Given a seed track, the system finds similar tracks via embeddings and collaborative patterns, then uses contextual bandits to sequence them based on listener reactions in real time.([Beats To Rap On][5])

### 4.5 Taste Profile & “Exclude from your taste profile”

Spotify recently added explicit controls like **“Exclude from your Taste Profile”** and track-level exclusions: if you turn this on, plays of that content contribute less (or not at all) to the profile used for recommendations.([Spotify Engineering][12])

That’s an attempt to separate “guilty pleasure / background listening / kids using your account” from your core taste representation.

---

## 5. Discovery Mode and the “payola layer”

The biggest recent change in how the algorithm works **from an artist’s perspective** is Discovery Mode.

### 5.1 What Discovery Mode is

* Artists or labels can **opt certain tracks into Discovery Mode** via Spotify for Artists or distributor dashboards.
* In exchange for a **lower royalty rate (around 30% reduction)** on streams generated from certain contexts (Radio, Autoplay, personalized algorithmic surfaces), Spotify will treat a “Discovery Mode” flag as a **strong positive signal** when ranking tracks.([Chartlex][13])

Spotify says it’s not a guarantee of placement, but a “signal among many” that increases eligible exposure.

### 5.2 Criticism and current legal fights

Critics argue Discovery Mode is essentially **payola in algorithmic form**:

* Listeners are not clearly told which tracks are boosted for a royalty discount.
* Independent artists feel pressured to trade revenue for visibility in a system where visibility is already scarce.
* Investigations and reviews (and now lawsuits) compare Discovery Mode to old radio pay-for-play arrangements.([The Guardian][14])

In November 2025, Spotify was hit with a **class action lawsuit** alleging Discovery Mode is a deceptive “pay-for-play” scheme that misleads users by presenting paid boosts as purely “personalized” recommendations. Spotify has publicly called the claims “nonsense” and denies wrongdoing.([Forbes][15])

Books like Liz Pelly’s *Mood Machine* and major reviews in The Guardian, The New Yorker, Vulture, and others frame Discovery Mode as part of a broader system where:([The New Yorker][16])

* Playlist-centric listening erodes artist–listener relationships.
* Playlist slots become scarce, monetized real estate.
* Background-friendly “Spotify-core” music and ghost/AI-generated artists proliferate to feed mood playlists at low cost.

So today, **the algorithm is not just technical**; it’s entangled with contested economic and ethical mechanisms.

---

## 6. What actually “matters” to the algorithm

Even though weighting details are secret, multiple reverse-engineering efforts, industry guides, and Spotify’s own hints converge on a few key KPIs for whether a track is considered successful for a particular audience:([Beats To Rap On][5])

1. **Skip rate & early abandonment**

   * Fast skips (within first 5–10 seconds) are a strong negative signal.
   * Skipping multiple tracks in a row from a source (playlist, Radio, etc.) is bad for that source’s ranking.

2. **Completion and repeat-listen rate**

   * Tracks listened to all the way through — especially repeatedly — are treated as strong positive signals.

3. **Saves and playlist adds**

   * Adding to Library or playlists is a very high-value signal, especially when:

     * Added to many different users’ personal playlists.
     * Added early in a track’s life.

4. **Artist follows and catalog depth**

   * If listeners go on to explore more songs by the same artist, that strengthens the perceived “fit.”

5. **Session-level impact**

   * Whether your track extends sessions, reduces churn mid-session, or leads to “back” presses or app exits.

6. **Contextual performance**

   * A track might perform well in “Focus” playlists but poorly in “Party” — the algorithm learns *where* it works, not just if it works.

7. **Network effects**

   * Appearing alongside certain other tracks in many user playlists creates graph structure that collaborative filtering exploits.

Discovery Mode, Marquee, Showcase, and other promo tools effectively **tilt the playing field** in this environment by forcing the algorithm to consider certain tracks more often or score them differently in relevant contexts.([The Guardian][14])

---

## 7. Practical implications (especially if you’re an artist)

If your goal is to “work with” the Spotify ecosystem rather than fight it, all of this suggests a few actionable principles (none of which require gaming the system or violating TOS):

### 7.1 Optimize for *real* engagement, not vanity streams

* Early listeners matter a lot. Getting the track in front of listeners who are **likely to save, replay, and add to playlists** is more valuable than spraying it everywhere.
* Avoid:

  * Click farms
  * Artificial streams
  * Playlist manipulation scams
    These risk triggering fraud systems and can tank your track’s trust score.

### 7.2 Front-load the hook and reduce early drop-off

* The first 5–10 seconds are critical for skip behavior.
* Consider:

  * Getting to the hook faster
  * Avoiding very long intros on “discovery” versions of songs
  * Making the first sound “on-brand” for the mood your listeners expect

### 7.3 Use Spotify for Artists metadata carefully

* Fill in the pitch form thoroughly: genre, mood, culture, instruments, what playlists it might fit.
* Think in **playlist use-cases** (study, gym, sleep, commute) because the system and editors do.([music-tomorrow.com][7])

### 7.4 Sequence releases and build a coherent catalog

* Regular releases help the algorithm continually test you with small exploration slices.
* Consistent sonic identity can make your catalog easier to cluster in embedding space.

### 7.5 Consider Discovery Mode with eyes open

* For some tracks, the trade-off (lower royalty per stream for more exposure) might make sense; for others, it may be a net negative.
* Given ongoing legal and ethical scrutiny, understand that:

  * Participation might impact perceptions (fans / peers).
  * Not participating might put you at a relative disadvantage in certain algorithmic contexts.([Making A Scene!][17])

### 7.6 Use Taste Profile controls as a listener

If you’re a creator but also a heavy listener:

* Use “Exclude from your Taste Profile” when listening to unrelated stuff (kids’ songs, white noise, etc.) so it doesn’t pollute your recommendations.([Spotify Engineering][12])

---

## 8. Open questions & research frontiers

For a “deep research” mindset, here are active or emerging areas:

* **Fairness & bias**

  * Are major-label tracks over-favored even controlling for quality and listener behavior?
  * Do non-English or niche genres systematically get less exploration?
* **AI-generated music and ghost artists**

  * Platforms filling mood playlists with cheap or first-party content that competes with human artists.([The Guardian][18])
* **Regulation & transparency**

  * Ongoing investigations, lawsuits, and calls for platforms to label paid promotion and explain recommendation logic to users.([Forbes][15])
* **User autonomy**

  * How far should “personalization” go before it becomes manipulation? Books like *Mood Machine* argue we’re already past that line.([The Guardian][18])

---

If you tell me your angle (e.g., “I’m an indie artist trying to grow on Spotify,” “I want to design a competing recommender,” or “I’m teaching a class on this”), I can follow up with a more targeted, step-by-step playbook or a technical deep dive into the ML side (models, embeddings, bandits, online A/B testing).

* [The Verge](https://www.theverge.com/2025/1/16/24344429/spotify-streaming-book-mood-machine?utm_source=chatgpt.com)
* [The Guardian](https://www.theguardian.com/music/article/2024/jul/29/the-music-industry-is-engineering-artist-popularity-and-listeners-are-right-to-be-angry?utm_source=chatgpt.com)
* [The Verge](https://www.theverge.com/2024/11/25/24305959/drake-kendrick-umg-spotify-not-like-us-legal-action?utm_source=chatgpt.com)

[1]: https://www.spotify.com/safetyandprivacy/understanding-recommendations?utm_source=chatgpt.com "safetyandprivacy/understanding-recommendations"
[2]: https://publications.lib.chalmers.se/records/fulltext/256144/256144.pdf?utm_source=chatgpt.com "Music Recommendations Based on Real-Time Data"
[3]: https://newsroom.spotify.com/2023-10-18/how-spotify-uses-design-to-make-personalization-features-delightful/?utm_source=chatgpt.com "How Spotify Uses Design To Make Personalization ..."
[4]: https://newsroom.spotify.com/2025-10-01/exclude-tracks-taste-profile/?utm_source=chatgpt.com "Spotify Gives You Even More Control With the Ability to ..."
[5]: https://beatstorapon.com/blog/ultimate-guide-to-spotify-music-algorithm/?utm_source=chatgpt.com "Spotify's Music Recommendation Algorithm: The Complete ..."
[6]: https://www.jatit.org/volumes/Vol99No23/1Vol99No23.pdf?utm_source=chatgpt.com "THE BEST METHOD OF MUSIC RECOMMENDATION ..."
[7]: https://www.music-tomorrow.com/blog/how-spotify-recommendation-system-works-complete-guide?utm_source=chatgpt.com "Inside Spotify's Recommendation System: A Complete ..."
[8]: https://pyimagesearch.com/2023/10/30/spotify-music-recommendation-systems/?utm_source=chatgpt.com "Spotify Music Recommendation Systems"
[9]: https://assets.amazon.science/2e/64/c111ff2c485c8b2f1d57f99fa549/a-scalable-model-for-online-contextual-music-recommendations.pdf?utm_source=chatgpt.com "a-scalable-model-for-online-contextual-music- ..."
[10]: https://patentpc.com/blog/how-spotifys-patents-are-reshaping-personalized-music-algorithms?utm_source=chatgpt.com "How Spotify's Patents Are Reshaping Personalized Music ..."
[11]: https://newsroom.spotify.com/2021-10-13/adding-that-extra-you-to-your-discovery-oskar-stal-spotify-vice-president-of-personalization-explains-how-it-works/?utm_source=chatgpt.com "Oskar Stål, Spotify Vice President of Personalization ..."
[12]: https://engineering.atspotify.com/2023/10/exclude-from-your-taste-profile?utm_source=chatgpt.com "Exclude from Your Taste Profile - Spotify Engineering"
[13]: https://chartlex.com/blogs/news/spotify-discovery-mode-explained-indie-artists?srsltid=AfmBOorZkcvrjcB8-QPAogSnGIR2thMDG7NidmKESeWLwbOovgypFGD7&utm_source=chatgpt.com "Spotify Discovery Mode Explained: Is It Worth It for Indie ..."
[14]: https://www.theguardian.com/music/2025/feb/19/spotify-discovery-mode-payola-playlist?utm_source=chatgpt.com "Pay to get playlisted? The accusations against Spotify's Discovery Mode"
[15]: https://www.forbes.com/sites/conormurray/2025/11/05/spotify-hit-with-class-action-lawsuit-alleging-discovery-mode-is-a-pay-for-play-scheme/?utm_source=chatgpt.com "Spotify's Discovery Mode Is A 'Pay-For-Play Scheme,' ..."
[16]: https://www.newyorker.com/magazine/2024/12/30/mood-machine-liz-pelly-book-review?utm_source=chatgpt.com "Is There Any Escape from the Spotify Syndrome?"
[17]: https://www.makingascene.org/spotifys-discovery-mode-the-new-payola-hurting-indie-artists/?utm_source=chatgpt.com "Spotify's Discovery Mode: The New Payola Hurting Indie Artists"
[18]: https://www.theguardian.com/books/2025/mar/05/mood-machine-by-liz-pelly-review-a-savage-indictment-of-spotify?utm_source=chatgpt.com "Mood Machine by Liz Pelly review - a savage indictment of Spotify"

The short version:
Playlists absolutely can move the needle, but *which* playlist you land on matters more than *how many* you collect. A “good” playlist is one where your track gets real, engaged human listening that feeds the algorithm. A “bad” playlist is one where you rack up empty streams (or fraud) that don’t convert into saves, followers, or algorithmic love—and can even get you penalized.

I’ll break this into:

1. What we actually know about the **effect of playlist placement**
2. **Types** of playlists and how they behave
3. Concrete **features of a good playlist**
4. Clear **red flags of bad / harmful playlists**
5. A practical **checklist for evaluating playlists** in Spotify for Artists

---

## 1. What we know about the effect of getting on playlists

### Playlists are a big share of listening

* Industry estimates and Spotify-facing analytics tools suggest roughly **one-third or more** of listening time on Spotify happens via playlists, not albums or direct plays.([RattleDistro][1])
* One analytics provider (Soundcharts) estimates **user-generated playlists alone account for ~36% of content hours**, about 2.5× the imprint of curated editorial playlists.([Soundcharts][2])

So, *in principle*, playlists are a major discovery channel.

### Case-study effects: from bumps to breakouts

From recent case studies and platform-facing blogs:

* iMusician reports editorial placements producing **~160% stream increases in week one**, with examples like:

  * ~70,000 streams during an editorial placement, followed by **700,000+ additional algorithmic streams afterward** as the track was picked up by Discover Weekly, Radio, etc.([iMusician][3])
* Multiple marketing breakdowns show that what really matters is **what happens after** the initial playlist bump:

  * Tracks that convert those playlist listens into **saves, playlist adds, and followers** tend to be picked up by algorithmic playlists later.([D4 Music Marketing][4])

So playlists have **two layers of impact**:

1. **Direct**: streams while you’re on the playlist.
2. **Indirect**: engagement from that playlist feeds the algorithm, which can unlock Discover Weekly, Release Radar, Radio/Autoplay, etc.

If playlist streams come with high skips and no saves/followers, you often get **only the direct effect**, and sometimes even a negative algorithmic signal.

---

## 2. Types of playlists and why they behave differently

On Spotify there are four main buckets, which behave very differently for you as an artist:

1. **Editorial playlists (Spotify-branded, human + algorithm)**

   * Made or at least seeded by Spotify staff (e.g., New Music Friday, RapCaviar, Deep Focus).
   * Often *algotorial*: editors choose a pool, algorithms personalize ordering and track variants per user.([Spotify Engineering][5])
   * Big potential reach but also high skip risk if you’re not a perfect fit.

2. **Algorithmic playlists (personalized)**

   * Discover Weekly, Release Radar, Daily Mixes, etc.([Stereofox Music Blog][6])
   * Powered heavily by **save rate, skip rate, repeat listens, and playlist adds**. By mid-2025, several analyses converge on these as the four key metrics.([artist.tools][7])
   * A good run on smaller playlists is often what unlocks these.

3. **Independent / third-party curated playlists**

   * Run by individuals, blogs, brands, labels, etc.
   * Accessible and numerous; case studies show they can have a **huge impact on streams and visibility** when genuine.([Feature.fm Blog][8])

4. **User-generated / personal playlists (“the long tail”)**

   * Millions of small personal playlists; collectively they represent a *massive* share of listening (again, ~36% of content hours).([Soundcharts][2])
   * You don’t “pitch” these, but when many individuals organically playlist you, it’s a very strong, trustable signal for the algorithm.

Takeaway: **editorial and algorithmic playlists are powerful, but the long tail of independent and personal playlists is where sustainable, less volatile growth comes from.**

---

## 3. Features of a *good* playlist (from the artist’s POV)

Let’s define “good” as: *it grows your track in a way the algorithm likes and doesn’t put your catalog at risk.*

### 3.1 Real, engaged human audience

This is the single most important feature.

Indicators:

* **Reasonable follower count to stream ratio.**

  * If a playlist has, say, 5k followers and is delivering 3–10k streams per track over a month, that’s plausible.
  * If it has 500 followers and somehow generates 100k streams per track, that’s suspicious.

* **Normal geographic patterns.**

  * In Spotify for Artists, your streams from that playlist map to countries and cities that make sense for your genre/marketing.
  * If you’re an indie folk act from the US and 90% of streams come from a cluster of tiny markets you’ve never touched, that’s a red flag.

* **Reasonable listening time.**

  * Spotify explicitly warns against artificial streams and third-party services that guarantee placement/streams. Using them can result in **removal or penalties**.([Spotify][9])
  * Recent reports: Spotify has removed ~75 million “spammy” or AI-generated tracks and is ramping up spam detection. Fraud around playlists is a big part of that crackdown.([The Guardian][10])

A playlist is good if you can see: *streams + saves + followers*, not just streams.

### 3.2 Strong per-track engagement metrics

You can measure “playlist quality” indirectly through **how your track behaves** on it.

Key metrics (pulled from Spotify-facing analytics and recent 2024–25 breakdowns):([Kabir Sehgal][11])

* **Save rate / Save-to-stream ratio**

  * Tracks with **save-to-listen ratios above ~20%** are *far* more likely to be picked up by algorithmic playlists.([smart-noise.com][12])
  * A “good” playlist is one where your save rate is *higher* than your baseline, not lower.

* **Skip rate (especially in the first 30 seconds)**

  * High early skips are an algorithmic killer. If a playlist’s audience regularly bails on your track in 5–10 seconds, that playlist may be a bad context for you.([SoundCampaign][13])

* **Repeat listens & streams per listener**

  * More than 1 stream per listener indicates people actually liked what they heard and came back.
  * Several 2025 guides identify **repeat listens** as one of the four core metrics driving algorithmic growth.([artist.tools][7])

* **Playlist additions vs listener increase**

  * Kabir Sehgal suggests tracking whether increased listeners from playlists translate into long-term listener growth and followers. If a playlist adds a ton of streams but barely moves your listener/follower base, its quality is questionable.([Kabir Sehgal][11])

A playlist is “good” if:

> Streams from it come with **high saves, low skips, elevated repeat rates, and noticeable follower growth**.

### 3.3 Clear, coherent context (mood, genre, use-case)

Spotify’s own engineering posts emphasize that playlists built around **moods and activities** perform well (“Chillin’ on a Dirt Road,” “my life is a movie,” “Classic Road Trip Songs,” etc.).([Spotify Engineering][5])

Good playlists:

* Have a **clear identity** (e.g., “Ambient Piano for Sleep,” “Modern East Coast Boom Bap,” “Melancholy Synthwave for Night Drives”).
* Maintain a **cohesive sonic and emotional palette**, which:

  * Keeps skip rate low,
  * Makes listeners trust the curator,
  * And increases the chance your track is the “right song in the right context.”

Random “all-genre” or extremely broad “Hits 2025” lists often produce lower engagement per track unless they’re truly massive.

### 3.4 Reasonable length & position dynamics

Case studies and analytics posts highlight that a track’s **position** and the **length** of the playlist heavily influence impact:([iMusician][3])

* Shorter to medium-length playlists (say **30–150 tracks**) give each song a fair shot.
* If a playlist has **thousands of tracks**, most songs near the bottom get almost no real exposure.
* Being consistently in the **top 20–40 slots** usually yields far more streams than being buried.

Good playlists:

* Aren’t “dumpster” lists of 1,000+ songs.
* Either:

  * Rotate tracks but keep a manageable length, or
  * Maintain a stable core and update carefully.

### 3.5 Network and reputation effects

Some playlists are plugged into bigger ecosystems:

* Editorial and major algorithmic lists are clearly powerful.([Stereofox Music Blog][6])
* Good independent curators often:

  * Run multiple playlists,
  * Have cross-promotion on TikTok/IG/YouTube, and
  * Have track records of **moving artists into algorithmic playlists afterwards**.([Steve Cardigan][14])

A “good” playlist often comes attached to:

* An identifiable curator (site, socials, brand).
* A history of breaking songs or at least consistently driving *quality* engagement.

---

## 4. Red flags: what makes a playlist *bad* (or outright dangerous)

### 4.1 Violates Spotify’s rules (guaranteed placement, paid streams)

Spotify is very explicit:

* Any third-party that **guarantees playlist placement or streams** is violating their terms.([Spotify][9])
* Using such services can result in:

  * Track removal,
  * Royalty clawbacks,
  * Or even account/distribution penalties.

Recent news + threat-intel reports show:

* AI-powered streaming fraud schemes generating *millions* of fake streams and facing serious legal consequences.([WIRED][15])
* Spotify has removed **75m+ spammy tracks**, with new AI-based detection and spam filters targeting manipulation.([The Guardian][10])

If a playlist or service sells itself on “X streams guaranteed,” assume it’s a bad playlist from an algorithmic and risk perspective.

### 4.2 Obvious fraud patterns in your data

Signs from Spotify for Artists that a playlist is low-quality or fraudulent:

* Streams spike unrealistically fast and then drop to almost zero the moment you leave the playlist.
* **Geography makes no sense** (e.g., 80–90% of your streams suddenly coming from countries with no other presence in your catalog).
* **Very short listening durations**, high early skips, almost no saves or followers despite huge stream counts.([Reddit][16])

That’s classic “numbers look big, value is zero” behavior.

### 4.3 Massive “dump” playlists with low engagement

Some playlists exist just to sell slots to as many artists as possible:

* Thousands of tracks, minimal followers.
* Tracks constantly rotated in and out.
* Streams per track are tiny or highly uneven.

These might not be fraudulent, but they’re **bad** in the sense that they:

* Don’t generate meaningful listening.
* Don’t drive saves or followers.
* Don’t send a strong positive signal to the algorithm.

### 4.4 Context-mismatch and “background fodder”

Given Spotify’s move toward mood/background playlists and even “ghost artists” / stock music for cheap ambience, you can end up in contexts where you just don’t fit.([The Times][17])

When that happens:

* Listeners skip you because your vibe breaks the flow.
* Your track’s performance in those contexts looks poor, which can drag your overall metrics down.

If your song is, say, lyrically dense and energetic, dropping it into a “Lo-Fi Beats / Deep Focus” style playlist may do more harm than good.

---

## 5. Practical checklist: how to judge if a playlist is “good” or “bad”

Here’s a concrete way to evaluate a playlist opportunity or a placement you already have.

### Step 1: Inspect the playlist itself (before saying yes)

Ask:

1. **Identity & curation**

   * Does it have a clear mood/genre/use-case?
   * Does the tracklist look coherent, or is it random?

2. **Follower & track counts**

   * Followers: Enough to matter, but not absurd relative to engagement claims.
   * Track count: Under ~150–200 is healthier; 1,000+ is usually low-value.

3. **Curator transparency**

   * Can you find the curator on IG/website?
   * Do they talk about community, curation, or is it all “buy slots, get streams”?

4. **No guaranteed streams**

   * Any “we guarantee 10k+ streams” language = walk away.([Spotify][9])

### Step 2: After you’re added, watch these numbers in Spotify for Artists

For that specific playlist (or the “Playlists” tab filtered appropriately), check:

1. **Save rate**

   * Is the save-to-stream ratio **> your usual baseline**?
   * If it’s **< 10–15%** while your organic channels are higher, that’s a bad sign.([smart-noise.com][12])

2. **Skip rate / completion**

   * Are listeners finishing the song or bailing early?
   * If the playlist has much *worse* skip behavior than your direct/album plays, it’s the wrong audience.

3. **Streams per listener**

   * > 1 stream/listener suggests some replay value.
   * If it’s close to exactly 1 and never rises, that can indicate low emotional impact.

4. **Follower growth**

   * Did your **follower count and monthly listeners** climb during the placement?([Kabir Sehgal][11])

If the answer to most of these is “yes”, this playlist is probably good for you—even if it’s small. If it’s “no”, it might be vanity.

### Step 3: Watch what happens *after* the playlist

The real test of a strong playlist is what happens **once you leave**:

* Do you see an uptick in **Discover Weekly, Release Radar, Radio** streams in the weeks following placement?

  * Multiple analyses note that high save and engagement rates from early listeners are what push tracks into algorithmic playlists.([Musosoup][18])
* Do your **daily streams settle at a higher baseline** than before, or do you collapse back to zero?

Good playlists help your track “graduate” into the wider recommendation system. Bad ones give you a spike, then silence—or worse, they poison your metrics.

---

If you tell me how you’re releasing music (solo indie artist, label, AI-driven catalog, etc.), I can turn this into a tailored strategy:

* how many playlist pitches to aim for per release,
* how to prioritize editorial vs indie curators vs algorithmic triggers, and
* how to set up a simple dashboard (even just in a spreadsheet) to score every playlist you land on.

- [The Guardian](https://www.theguardian.com/music/2025/sep/25/spotify-removes-75m-spam-tracks-past-year-ai-increases-ability-make-fake-music?utm_source=chatgpt.com)
- [Music Business Worldwide](https://www.musicbusinessworldwide.com/spotify-has-deleted-75m-spammy-tracks-as-it-unveils-new-ai-music-policies/?utm_source=chatgpt.com)
- [WIRED](https://www.wired.com/story/ai-bots-streaming-music?utm_source=chatgpt.com)
- [El País](https://elpais.com/icon/2025-11-23/el-caso-de-drake-y-las-millones-de-escuchas-falsas-la-demanda-contra-spotify-que-cuestiona-a-la-industria-musical.html?utm_source=chatgpt.com)
- [The Guardian](https://www.theguardian.com/music/2025/feb/19/spotify-discovery-mode-payola-playlist?utm_source=chatgpt.com)
- [The Times](https://www.thetimes.co.uk/article/spotifys-shadowy-ghost-artists-unmasked-hstc9wcdb?utm_source=chatgpt.com)
- [The Verge](https://www.theverge.com/2025/1/16/24344429/spotify-streaming-book-mood-machine?utm_source=chatgpt.com)

[1]: https://rattledistro.com/blog/the-importance-of-playlist-pitching?utm_source=chatgpt.com "The Importance of Playlist Pitching to Spotify: Elevate Your ..."
[2]: https://soundcharts.com/spotify-analytics?utm_source=chatgpt.com "Spotify Artists Analytics: Track Playlists, Charts, & Audience ..."
[3]: https://imusician.pro/en/resources/blog/do-playlists-really-boost-streams?utm_source=chatgpt.com "Do Playlists Boost Streams?"
[4]: https://d4musicmarketing.com/improve-song-performance-spotify-algorithm/?utm_source=chatgpt.com "How to Improve Your Song's Performance in Spotify's ..."
[5]: https://engineering.atspotify.com/2023/04/humans-machines-a-look-behind-spotifys-algotorial-playlists?utm_source=chatgpt.com "Humans + Machines: A Look Behind the Playlists Powered by ..."
[6]: https://www.stereofox.com/articles/the-different-kinds-of-spotify-playlists-explained/?utm_source=chatgpt.com "The Different Kinds of Spotify Playlists Explained"
[7]: https://www.artist.tools/post/master-spotify-algorithmic-playlists?utm_source=chatgpt.com "Master Spotify Algorithmic Playlists"
[8]: https://blog.feature.fm/discover-the-impact-of-third-party-playlists-on-an-independent-artists-streams/?utm_source=chatgpt.com "Case Study: discover the impact of third party playlists on ..."
[9]: https://support.spotify.com/us/artists/article/third-party-services-that-guarantee-streams/?utm_source=chatgpt.com "Artificial streaming and paid 3rd-party services that ..."
[10]: https://www.theguardian.com/music/2025/sep/25/spotify-removes-75m-spam-tracks-past-year-ai-increases-ability-make-fake-music?utm_source=chatgpt.com "Spotify removes 75m spam tracks in past year as AI increases ability to make fake music"
[11]: https://kabir.cc/focus-on-these-5-metrics-on-spotify-for-artists/?utm_source=chatgpt.com "Focus on these 5 metrics on Spotify for Artists - Kabir Sehgal"
[12]: https://smart-noise.com/spotify-saves/?utm_source=chatgpt.com "Spotify Saves Guide: How to Boost Your Tracks in 2025"
[13]: https://soundcamps.com/blog/spotify-algorithm-explained/?utm_source=chatgpt.com "Spotify Algorithm Explained: 10x Your Streams"
[14]: https://stevecardigan.substack.com/p/spotifys-algorithm-and-the-unbelievable?utm_source=chatgpt.com "Spotify's Algorithm and the unbelievable value of \"curator ..."
[15]: https://www.wired.com/story/ai-bots-streaming-music?utm_source=chatgpt.com "'A Billion Streams and No Fans': Inside a $10 Million AI Music Fraud Case"
[16]: https://www.reddit.com/r/musicmarketing/comments/qi68sh/manipulating_spotify_streams_with_streaming_fraud/?utm_source=chatgpt.com "Manipulating Spotify Streams with Streaming Fraud"
[17]: https://www.thetimes.co.uk/article/spotifys-shadowy-ghost-artists-unmasked-hstc9wcdb?utm_source=chatgpt.com "Spotify 'ghost artists': they're on your playlist but are they real?"
[18]: https://musosoup.com/blog/spotify-algorithmic-playlists?utm_source=chatgpt.com "Spotify Algorithmic Playlists: How to Get on Release Radar ..."


You can absolutely build a “playlist rating bot,” but you want to be very deliberate about what you’re scoring and what data you actually have access to.

I’ll treat this as a design/spec problem and give you a concrete framework:

* What data you can realistically pull
* Features that map to “good vs bad” (0–1 quality score)
* Features that map to **reach**
* A scoring formula and example weights
* How this would look in an actual bot / pipeline

I’ll flag where I’m inferring vs where there’s external evidence from existing analyzers and Spotify docs.

---

## 1. Data you can realistically use

### 1.1 From Spotify Web API (public)

From the Web API docs, for each playlist you can get:([Spotify for Developers][1])

* `name`, `description`
* `followers.total` (total followers)
* `owner` (user vs Spotify, display name, id)
* Public vs private, collaborative flag
* `tracks.items`:

  * Track IDs, artists, album, added_at, added_by
* For each track (via separate calls):

  * Popularity (0–100)
  * Release date
  * Audio features (energy, danceability, valence, tempo, etc.)

Important TOS constraint: the Web API explicitly forbids using Spotify content to train general ML/AI models. So your bot should either be rule-based / simple ML on *derived metrics*, or you’ll need to be very careful not to build a general-purpose model on raw catalog data.([Spotify for Developers][2])

### 1.2 From your own data / Spotify for Artists

There’s no public Spotify for Artists API, but the insights they expose show what *kinds* of metrics matter:([Spotify for Artists][3])

* Streams per playlist
* Listeners per playlist
* Saves, save rate
* Completion rate / skip rate
* Streams per listener / repeat listens
* Playlist adds

These four keep coming up in 2024–25 analyses as the core engagement metrics for algorithmic performance:([smart-noise.com][4])

1. Save rate
2. Repeat listens
3. Playlist adds
4. Skip / completion rate

If you want your bot to rate “good/bad *for me* as an artist,” you should ingest these metrics (even manually via CSV exports) and join them to playlists.

### 1.3 From third-party playlist analyzers

Existing tools already compute “is this playlist good or scammy?” using a mix of public and historical data:

* **artist.tools** – bot detection, historical follower growth, avg track age, contact info, listener estimates.([artist.tools][5])
* **IsItAGoodPlaylist.com** – exposes follower growth curves, engagement patterns; they explicitly highlight:

  * Sudden follower spikes or drops
  * Followers vs track engagement mismatch
    as bot/fraud indicators.([Is It A Good Playlist][6])
* **MusoSoup / others** – talk about engagement rates, demographics, playlist dynamics as core metrics.([Musosoup][7])

You can either:

* Call their APIs (IsItAGoodPlaylist advertises API access),([Is It A Good Playlist][8])
* Or replicate the same logic by storing daily snapshots of playlist followers and recomputing growth patterns yourself.

---

## 2. What “good vs bad” should actually mean

You want a **[0,1] quality score** that answers:

> “If I put my track here, how likely am I to get *real, healthy engagement* versus fake streams or useless vanity numbers?”

That breaks down into:

1. **Engagement quality** – when people hear *my* song on this playlist, do they save, repeat, follow, etc.?
2. **Structural quality** – is the playlist well-built (coherent, not a 5,000-track dump, not 80% one artist)?
3. **Authenticity / fraud risk** – is it likely to be botted or violating Spotify’s anti-artificial-streaming policies?([Spotify for Artists][9])

Your bot’s “good/bad” score should be a composite of those three pillars.

---

## 3. Feature set for the **quality score** (0–1)

I’ll lay this out as if we’re designing features for a model, but you can also plug them into a hand-crafted formula.

### 3.1 Engagement features (best if you have Spotify for Artists data)

For playlists that already contain your tracks:

* **Save rate on this playlist (SR_p)**
  `SR_p = saves_from_playlist / streams_from_playlist`

  * Normalize by comparing to your *global* save rate.
  * Quality signal: playlists where SR_p is > your median are “good for you.”([smart-noise.com][4])

* **Skip / completion rate (CR_p)**

  * Completion: share of streams reaching ~90–100% of track.
  * High early skip rate is negative.
  * Normalize similarly vs your global completion rate.

* **Streams per listener (SPL_p)**

  * A proxy for repeat listens: SPL_p > 1.0 = some replay value.

* **Playlist adds triggered by this playlist (PA_p)**

  * How often listeners hearing you there then add you to *their* playlists.
  * This is one of the strongest signals that a playlist creates durable fans vs shallow background streams.

You can turn these into a [0,1] engagement sub-score for a playlist P:

```text
Q_engage(P) = clamp( 
    0.35 * z_tanh(SR_p) +
    0.25 * z_tanh(CR_p) +
    0.20 * z_tanh(SPL_p) +
    0.20 * z_tanh(PA_p),
  0, 1)
```

Where `z_tanh` is “take value relative to your median and map via tanh-ish squashing into [-1,1], then rescale to [0,1].”

If you don’t have per-playlist engagement, skip this term for new playlists and rely more on structural/authenticity priors.

### 3.2 Structural features (from Web API only)

These are observable for *any* playlist.

**Size & density**

* `track_count` – very large lists (e.g., > 500–1,000 tracks) tend to be low value per song.
* Define a size score:

```text
size_score = 
  1.0 if 30 <= track_count <= 150
  linearly decreasing outside that band, floored at 0
```

**Artist diversity and dominance**

Compute:

* `unique_artists / track_count`
* `max_artist_share` = max fraction of tracks by a single artist

Red flags:

* `max_artist_share > 0.5` (one artist dominates)
* `unique_artists / track_count < 0.2` (very low diversity)

Turn into:

```text
diversity_score = clamp(
   0.5 * (unique_artists / track_count) +
   0.5 * (1 - max_artist_share),
  0, 1)
```

**Popularity & discovery profile**

Using track popularity (0–100):([metrics.musicstax.com][10])

* `avg_popularity`
* `median_popularity`
* `popularity_std`
* Share of tracks with popularity < 20, 20–60, > 60.

Good playlists tend to have a **healthy mix**: some familiar tracks + some mid/low popularity tracks. A playlist of only ultra-obscure songs with huge follower counts is suspicious.

Define something like:

```text
mix_score = penalty if:
  (avg_popularity < 15 and followers > big_threshold)
otherwise reward mid-range mixture (20–60)
```

**Audio / mood cohesion**

From audio features:([smart-noise.com][4])

* Distributions of:

  * Energy, valence, danceability, tempo, loudness
* Cohesion measure:

  * Lower standard deviation in key mood features ⇒ more coherent.

You might compute:

```text
cohesion_score = 1 - normalized_stddev(energy, valence, tempo)
```

Then structural quality:

```text
Q_struct(P) = clamp(
  0.30 * size_score +
  0.25 * diversity_score +
  0.25 * mix_score +
  0.20 * cohesion_score,
  0, 1)
```

### 3.3 Authenticity / fraud-risk features

This is where you mirror what tools like artist.tools and IsItAGoodPlaylist are doing.([artist.tools][5])

You want a **risk score** in [0,1], where 1 = almost certainly legit, 0 = very likely botted. Then you’ll *multiply* quality by this.

Features:

1. **Follower growth pattern** (requires time series)

   * Sudden spikes: e.g., 1k → 50k followers in a week.
   * Big purges: sharp drops when Spotify removes fake accounts.([Is It A Good Playlist][6])
   * Legit growth tends to be smoother.

2. **Followers vs engagement mismatch**

   * If the playlist has huge followers but:

     * Most tracks have extremely low play counts overall, or
     * Your own tracks get almost no streams from it
       that’s a classic bot pattern.([Is It A Good Playlist][6])

3. **Curator profile & description**

   * Keywords in description or linked site like “pay for placement,” “guaranteed streams,” etc. are strong negative signals.([Spotify][11])
   * Curator profile followers vs playlist followers: if the playlist has 100k followers and the curator account has 20, that’s suspicious.([Symphonic Blog][12])

4. **Geography of streams (if you have it)**

   * Multiple fraud analyses note that bots cluster in specific VPN-available geos. Genuine playlists show a more natural, messy spread of locations.([The Guardian][13])

Define a risk/legitimacy score:

```text
Q_auth(P) = clamp(
  0.40 * smooth_growth_score +
  0.25 * follower_engagement_ratio_score +
  0.20 * curator_credibility_score +
  0.15 * geo_plausibility_score,
  0, 1)
```

Where:

* `smooth_growth_score` penalizes big spikes and purges
* `follower_engagement_ratio_score` penalizes extreme mismatch
* `curator_credibility_score` uses text heuristics & curator stats
* `geo_plausibility_score` penalizes weird, concentrated geos if you see them

---

## 4. Feature set for **reach**

You can treat reach as a separate number that you either:

* Output raw (followers, listener estimate), **or**
* Normalize to [0,1] relative to all playlists you track.

Basic components:

1. **Follower count (F)**
2. **Active listener estimate (L)** – if you can infer from streams/listeners
3. **Growth rate (G)** – recent follower growth, clipped
4. **Track exposure share** – for *your* track, how many daily/weekly listeners the playlist drives

A simple reach score:

```text
R(P) = clamp(
  0.50 * norm_log(F) +
  0.30 * norm_log(L) +
  0.20 * norm_pos(G),
  0, 1)
```

Where:

* `norm_log(F) = log(1+F) / log(1+F_max)` across your dataset
* `norm_pos(G)` normalizes recent weekly or monthly growth into [0,1] with clipping so insane spikes don’t dominate

You might *also* output raw `F` and `L_est` alongside the normalized `R(P)`.

---

## 5. Putting it together: the scoring formula

You want:

* **Quality score** `Q_final(P)` in [0,1]
* **Reach score** `R(P)` in [0,1] (plus raw numbers)

Combine:

```text
Q_base(P) = 
  w_eng * Q_engage(P) +
  w_struct * Q_struct(P)

# Example weights:
#   w_eng   = 0.6 (if you have engagement data)
#   w_struct = 0.4

Q_final(P) = clamp( Q_base(P) * Q_auth(P), 0, 1 )
```

Interpretation:

* `Q_base` says “how well this playlist works in theory and in practice for my music.”
* `Q_auth` downweights things that smell like fraud / payola / bots.

For new playlists with no engagement data:

* Set `w_eng` lower, `w_struct` higher.
* E.g. `w_eng = 0.2, w_struct = 0.8`, until you gather enough streams to compute Q_engage.

Then you can define a combined “priority” for your bot:

```text
priority(P) = Q_final(P) * R(P)
```

…and rank playlists descending by **priority** if your goal is “high quality + high reach.” You can also explore:

* High Q, low R → small but excellent niche playlists
* High R, mid Q → big but mixed-quality playlists
* Low Q → avoid / flag as risky

---

## 6. Bot / pipeline architecture

High-level sketch:

### 6.1 Data model

Tables or collections like:

* `playlists`:

  * id, name, description, owner_id, followers, is_spotify_owned, is_collaborative, last_crawled
* `playlist_tracks`:

  * playlist_id, track_id, added_at, added_by
* `tracks`:

  * id, artist_ids, album_id, popularity, release_date, audio_features…
* `playlist_timeseries`:

  * playlist_id, date, followers, maybe listener_estimate (if you have it)
* `artist_engagement` (for you or your label roster):

  * track_id, playlist_id, date_range, streams, listeners, saves, skip_rate, completion_rate, playlist_adds

### 6.2 Crawler / updater

* A scheduled job that:

  * Takes a list of playlists (from your submissions, searches, 3rd-party APIs)
  * Calls Spotify Web API to:

    * Update playlist metadata & followers
    * Fetch track list and audio features
  * Appends a point to `playlist_timeseries` for growth analysis

Optional:

* Call external analyzers (artist.tools / isitagoodplaylist API) to pull bot flags, external growth curves, curator contact.

### 6.3 Scoring service

A service (could be a Python module or microservice) that:

1. Loads features for a given playlist ID from your DB.
2. Computes:

   * Q_engage (if you have data)
   * Q_struct
   * Q_auth
   * R
3. Stores these in `playlist_scores`:

   * playlist_id, timestamp, Q_final, R, priority, breakdown

Then your “bot” interface (CLI, Slack bot, custom GPT, web dashboard) just queries `playlist_scores` and sorts.

### 6.4 Example queries you’d enable

* “Top 50 playlists with `Q_final >= 0.7` and `R >= 0.3` for my last single.”
* “Playlists where engagement is good (Q_engage > 0.8) but reach is small (R < 0.2)” → good targets to deepen relationships.
* “Playlists with `Q_auth < 0.4` that currently contain my tracks” → candidates to ask curators to remove you or to stop pitching to.

---

## 7. Edge cases & cautions

### 7.1 False positives for fraud

The industry is currently full of stories of legitimate artists getting punished by anti-fraud systems.([The Guardian][13])

So your bot:

* Should treat `Q_auth` as **probabilistic**, not definitive.
* Should label things like:

  * “High fraud risk, verify manually”
  * Rather than “This playlist is fake.”

### 7.2 TOS & ethics

* Don’t scrape or retain data in a way that violates Spotify’s Web API terms (especially around training ML on content).([Spotify for Developers][2])
* Don’t build or sell a tool that implicitly encourages artificial streaming (e.g., “find botted playlists to juice your numbers”). Position it as **risk management** and **quality targeting**.

---

## 8. If you want, I can go one step more concrete

If you’d like next, I can:

* Sketch a **Python-style scoring function** (pseudo-code using `spotipy`-like calls) that implements the Q_struct + R(P) side; and
* Give you a draft schema for the `playlist_scores` table plus some sample threshold values (e.g., where to clip follower ranges, how to convert growth into 0–1).

But the core research piece you asked for is here:

* what signals the ecosystem already uses to call a playlist “good” or “botted,”
* how existing analyzers expose those signals,
* and how to turn them into a 0–1 quality score plus a normalized reach score your bot can compute.

[1]: https://developer.spotify.com/documentation/web-api/reference/get-playlist?utm_source=chatgpt.com "Get Playlist - Web API Reference"
[2]: https://developer.spotify.com/documentation/web-api/reference/search?utm_source=chatgpt.com "Web API Reference | Spotify for Developers"
[3]: https://artists.spotify.com/en/analytics?utm_source=chatgpt.com "Analytics – Spotify for Artists"
[4]: https://smart-noise.com/spotify-saves/?utm_source=chatgpt.com "Spotify Saves Guide: How to Boost Your Tracks in 2025"
[5]: https://www.artist.tools/features/playlist-analyzer?utm_source=chatgpt.com "Spotify Playlist Analyzer - Playlist Stats & Data"
[6]: https://www.isitagoodplaylist.com/article/the-best-spotify-playlist-analyzer-how-to-check-for-bots?utm_source=chatgpt.com "The Best Spotify Playlist Analyzer: How to Spot Fake ..."
[7]: https://musosoup.com/blog/spotify-playlist-analyzer?utm_source=chatgpt.com "Using a Spotify Playlist Analyser: How to Gain Insights and ..."
[8]: https://www.isitagoodplaylist.com/?utm_source=chatgpt.com "Is It A Good Spotify Playlist?"
[9]: https://artists.spotify.com/en/artificial-streaming?utm_source=chatgpt.com "Artificial Streaming"
[10]: https://metrics.musicstax.com/?utm_source=chatgpt.com "Musicstax Metrics: Spotify Popularity Score Tracker"
[11]: https://support.spotify.com/us/artists/article/third-party-services-that-guarantee-streams/?utm_source=chatgpt.com "Artificial streaming and paid 3rd-party services that ..."
[12]: https://blog.symphonic.com/2024/03/27/how-to-detect-fake-spotify-playlists-2/?utm_source=chatgpt.com "How To Detect Fake Spotify Playlists"
[13]: https://www.theguardian.com/music/2025/jun/03/ai-bot-farms-and-innocent-indie-victims-how-music-streaming-became-a-hotbed-of-and-fakery?utm_source=chatgpt.com "AI, bot farms and innocent indie victims: how music streaming became a hotbed of fraud and fakery"


