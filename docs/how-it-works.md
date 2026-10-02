# How L'archive works

[Back to the README](../README.md)

Everything below runs inside one Dataiku project. Each step is a recipe, a dataset or a folder that an engineer can open, read and rerun.

![The Dataiku Flow](../assets/screens/flow.jpg)

## 0. The recording goes in

A recording is dropped into a managed folder. For the demo, the archive holds six TED talks: three provided with the challenge (Prager 2015, Smith 2016, Buolamwini 2016) and three that we added (Bill Gates in 2015, 2020 and 2022). The three Gates talks are the same speaker on the same subject over seven years, which is what lets us test questions about time. Only the sound is used; the picture adds cost and was left for later.

## 1. It is cut into four-minute pieces

The files given at the start were already cut into pieces of exactly 240 seconds, 16 kHz, stereo. Every later step depends on that shape, so new audio is cut to the same shape rather than teaching each step a second format. The preparation recipe decodes the audio and writes the pieces: 74.7 minutes of audio in 35.8 seconds. It only adds files and never overwrites one, so rerunning it cannot destroy a recording.

A short piece also means the player never loads a two-hour file. A quote at minute 97 opens one four-minute piece at the right second.

## 2. Whisper writes down what was said

Dataiku's built-in speech recognition recipe returns the text of a file and nothing else: no timestamps. An answer that cannot point to a second cannot be played, so transcription is a Python recipe written by hand around faster-whisper, running on CPU inside Dataiku with no outside provider and no credit to run out.

Each sentence is written with its file, its piece number and its start and end inside that piece. Its second within the whole talk is the piece offset, (piece number minus 1) x 240 seconds, plus its start in the piece.

Accuracy, measured against TED's human subtitles on the three Gates talks: 5.5 % of 11,021 words differ. The subtitles are edited for reading, so part of that difference is TED's edit, and the figure is an upper bound.

## 3. Sentences become passages

One sentence is too short to answer a question; a whole file is too long to point at. Neighbouring sentences are joined until a passage can stand on its own: 203 passages for 1 h 41 min.

21 passages cross a four-minute cut. A passage cited at its start would send the reader to the wrong file for every line after the cut. So every line inside a passage carries its own marker, for example `[seg02 00:00]`, on the clock of the file it names, and the model must cite the line it quotes. A citation is as precise as a sentence, not as a passage.

## 4. Each passage is labelled and stored

Every passage carries three labels: the client, the mission and the real date of the recording. In the demo the clients are invented (CLIENT-A, CLIENT-B and internal sessions) and the dates are real, taken from each TED page. In production the client comes from the staffing system and the date from the meeting.

The passages are then turned into vectors with text-embedding-3-small and written to a Dataiku knowledge bank, so they can be found by meaning.

## 5. Someone asks, and the rules run first

**Who is asking.** Nobody logs in to the archive. The server asks Dataiku whose browser session it is and gets the login and the groups; the browser cannot choose them. The group gives the role: consultant, partner or administrator. An administrator can view the archive as a consultant to check the rules, and the journal still records the real account.

**What they may see.** The access rule runs on the server before the search. A passage outside the caller's clients is never scored, never ranked and never sent to a model. For a CLIENT-A consultant that removes 22 of the 203 passages; for CLIENT-B, 11. Filtering after the search would leave the best places filled with text that has to be thrown away, and one line of code between a consultant and another client's words.

**How old an answer may be.** Time is decided per question:

- A period is named ("in 2015", "the last two months"): recordings outside it are removed before scoring, as the access rule removes other clients.
- "The latest": a passage loses half its score for every two years it is older than the newest candidate.
- "How did it change": the best passage of each recording, laid out from oldest to newest.
- Nothing about time: age changes nothing.
- A subject recorded on several dates: the archive asks first, with the latest, what changed, one year, or everything.

## 6. The search

Two searches run on the allowed passages: one by meaning, comparing vectors, and one by exact words, a BM25 keyword search written by hand. The two ranked lists are fused by rank, and the four best passages go forward. The model reads four passages whether the archive holds 203 or a million, so the cost of a question does not grow with the archive.

## 7. The answer is written and checked

**Written.** The cheapest of the three models on the instance (gpt-5.6-luna) writes the answer by quoting lines, each with its marker. It was chosen for all 25 test questions; a route to a larger model for harder questions was built and never needed. An answer cost $0.0003 and took 2.3 seconds on average.

**Checked.** Every marker in the answer is matched against the markers of the passages actually retrieved. A model can write any string; matching it to a real marker is what separates a citation from something that looks like one. A verified citation gets a play button. One that matches nothing is shown in red and cannot be played.

**Heard.** The server streams the audio file straight out of the Dataiku folder with byte ranges, which is what lets the browser jump to a second, and sends the lines of that file with their times, so the text follows the voice.

**Refused.** If nothing in the archive answers, the archive says so. With the model, the refusal is decided by how far the closest passage is in meaning: answerable questions measured 0.66 to 1.09, an unanswerable one 1.65. A rank-fused score cannot do this job, because it is bounded and put an unanswerable question above an answerable one.

**Logged.** Every question and every reader's vote is written to a Dataiku folder, one file per event: who asked, under which profile, what came back, what the rule withheld, which model answered and what it cost.

## When the model ran out

On 28 September the provider account behind the instance's only model connection ran out of credit, and the instructor said it would not be restored. The gateway reported each failed call as a success; the backend reads the per-call status, so the failure was seen for what it was, and a circuit breaker stopped calling a dead model.

Since then the archive answers by exact words, under the same access and time rules, and quotes the passages as recorded, labelled "sans modèle". A passage is used only if it holds words carrying at least half of the question's weight, rare words weighing more; otherwise the answer is "not in the archive". The cost of that is in [the evaluation](evaluation.md): a question worded differently from what was said is refused.

## Our own meaning model

When the app starts, it builds a small meaning model from the transcripts alone: latent semantic analysis in numpy, 809 words after removing filler words, 16 dimensions. It needs no provider and costs nothing per question. It draws the map of the archive, names its 8 subjects from their most typical words, and proposes related passages above a similarity of 0.55. It never decides an answer.

![The map of the archive](../assets/screens/map.jpg)

## Reader feedback

Under each answer, the reader can mark it useful, outdated or wrong. "Outdated" asks which period they need and reruns the question on it in front of them. A passage marked outdated by two or more readers, and more often than useful, keeps half its score. It stays visible, and the person who owns it can review it.
