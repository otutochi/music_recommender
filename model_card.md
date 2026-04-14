# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

**VibeFinder 1.0**

---

## 2. Intended Use  

This system suggests 5 songs from a small catalog based on a user's preferred genre, mood, and energy level. It assumes the user knows what genre and mood they are in the mood for and can express a target energy level on a 0-to-1 scale. It is built for classroom exploration only — it is not designed for real users, production deployment, or commercial use.

---

## 3. How the Model Works  

The system looks at three things about each song: its genre, its mood, and its energy level. It compares those to the user's preferences and gives the song a score.

A song gets the most points (+2) for being in the user's favorite genre. It gets a smaller bonus (+1) for matching the user's preferred mood. Then it gets up to 1 extra point based on how close the song's energy is to what the user wants — the closer, the better. A song that matches on all three can score up to 4 points. The system scores every song in the catalog this way, sorts them from highest to lowest, and shows the top 5.

Every recommendation also comes with a plain-English explanation of why it scored the way it did, like "genre match (+2.0); mood match (+1.0); energy closeness (+0.97)."

---

## 4. Data  

The catalog contains 18 songs stored in a CSV file. We started with 10 songs and added 8 more to improve genre and mood diversity. The dataset now covers 15 genres (pop, lofi, rock, ambient, jazz, synthwave, indie pop, r&b, electronic, hip hop, classical, metal, funk, country, latin) and 14 moods (happy, chill, intense, relaxed, moody, focused, nostalgic, euphoric, melancholic, dreamy, aggressive, groovy, uplifting, romantic).

Most genres have only one song. Pop and lofi have the most representation (2–3 songs each). The data does not include any songs in languages other than English, and there are no lyrics, popularity scores, or release dates. The musical taste reflected in the catalog skews toward modern, Western, English-language music.

---

## 5. Strengths  

The system works well for users whose preferences align naturally with the catalog — a "happy pop" fan or a "chill lofi" listener gets results that feel intuitive and correct. It correctly separates two pop songs (Sunrise City vs Gym Hero) based on mood and energy differences, which shows the scoring logic can make meaningful distinctions within a genre. The explanation strings make every recommendation transparent — you can see exactly why a song ranked where it did, which is something most real apps do not offer. The simplicity of the scoring rule also makes it easy to experiment with weights and immediately understand the impact.

---

## 6. Limitations and Bias 

The system over-prioritizes genre because the genre match weight (+2.0) is larger than mood (+1.0) and energy closeness (max +1.0) combined, which means a song in the "right" genre will almost always outrank a better-fitting song in a different genre — creating a filter bubble where users only ever see one genre in their results. Most genres in the catalog (13 out of 15) have only a single song, so users who prefer funk, jazz, or classical have no real variety — the system has only one option to recommend regardless of their mood or energy preferences. Additionally, the scoring logic ignores acousticness and danceability entirely even though those features exist in the dataset and the user profile includes a `likes_acoustic` field, leaving acoustic-leaning and dance-focused users with no personalization on those dimensions. Finally, the energy closeness formula treats "too high" and "too low" the same way, so it cannot distinguish a user who wants "at least this much energy" from one who wants "exactly this energy level."

---

## 7. Evaluation  

We tested six user profiles — three "normal" profiles and three adversarial edge cases — and compared the top 5 results for each.

**Profiles tested:**

- **High-Energy Pop Fan** (pop, happy, energy 0.85) — Sunrise City ranked first with a near-perfect score of 3.97. Gym Hero also appeared but ranked lower because its mood is "intense," not "happy." This makes sense: the system correctly separated two pop songs by mood and energy, which is exactly what the scoring rule is designed to do.

- **Chill Lofi Listener** (lofi, chill, energy 0.35) — Library Rain scored a perfect 4.0 because it matches on genre, mood, and energy exactly. Midnight Coding came in second at 3.93, losing only 0.07 points from a slight energy gap. Both results feel right — these are the two most relaxing, low-key tracks in the catalog.

- **Deep Intense Rock** (rock, intense, energy 0.90) — Storm Runner was the clear winner at 3.99. Since it is the only rock song, the remaining four recommendations were non-rock songs sorted by mood and energy similarity. This exposed how the system falls back to weaker signals when the genre pool is thin.

**Surprising findings from edge cases:**

- **Conflicting preferences** (ambient, chill, energy 0.95) — The system recommended Spacewalk Thoughts (ambient, chill, energy 0.28) as #1 even though the user wanted high energy. Genre and mood matches (+3.0) completely overwhelmed the terrible energy fit (+0.33). This was the clearest example of genre dominance overriding a numerical preference.

- **Genre Outsider** (reggae, happy, energy 0.60) — No song in the catalog matched on genre, so the maximum possible score dropped to about 2.0. The system still produced reasonable results sorted by mood and energy, but every recommendation felt like a compromise rather than a confident pick.

- **Extreme Low Energy Metal** (metal, aggressive, energy 0.10) — Iron Thunder ranked first despite having energy 0.96 versus the user's target of 0.10. The genre and mood bonuses (+3.0) made the energy mismatch nearly irrelevant. This confirmed that the current weights make it almost impossible for energy to override a categorical match.

---

## 8. Future Work  

- **Use acousticness and danceability in scoring.** These features already exist in the dataset and the user profile has a `likes_acoustic` field, but the scoring function ignores them. Adding even a small weight for these would help differentiate users who want acoustic coffee-shop vibes from those who want electronic dance tracks.
- **Add a diversity penalty.** Right now, if a user likes pop, all top results could be pop songs. A penalty that reduces a song's score when the same genre or artist is already in the top results would create a more varied and interesting recommendation list.
- **Expand the catalog and add collaborative signals.** With only 18 songs, the system cannot meaningfully serve most genres. A larger dataset plus even simple collaborative filtering ("users who liked this also liked that") would reduce the cold-start problem for niche tastes.

---

## 9. Personal Reflection  

The biggest surprise was how much a single weight choice (genre at +2.0) shapes the entire output. It seemed like a small design decision, but it effectively locks users into one genre no matter what else they prefer. That made me realize that real recommendation apps are making hundreds of these weight decisions, and each one quietly steers what millions of people hear.

Building this also changed how I think about "why" a song shows up in my Discover Weekly. Before, I assumed it was magic. Now I see it as a scoring function — just a much bigger one with more features and more data. The explanations we built into this system ("genre match, mood match, energy closeness") are something I wish real apps would show, because it would make it much easier to understand and trust the recommendations.
