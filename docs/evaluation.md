# How L'archive was measured

[Back to the README](../README.md)

Every number in this repository comes from a run that wrote its results to a file. The files and the scripts that produced them are in the private repository.

## The test set

25 questions, written after reading the transcripts:

- **20 answerable**, each with the true line and its second written down beforehand.
- **5 unanswerable**: the answer is in none of the recordings, so the only right reply is "not in the archive". Two of them are traps, close in wording to something that was said.
- **3 access probes**: a CLIENT-A consultant asking about CLIENT-B's recording, which must return nothing from CLIENT-B.
- **8 questions about time**, added once the archive held the same speaker at three dates.

An answer counts as right when it gives the true answer and cites a line within 30 seconds of the true one.

## The two runs

| Measure | With the model, 24 Sep | Without it, 28 Sep |
|---|---|---|
| Archive | 3 talks, 47 passages, Whisper small | 6 talks, 203 passages, Whisper large-v3 |
| Right answer, cited within 30 s | 19 of 20 | 7 of 20 |
| Wrongly refused | 0 of 20 | 8 of 20 |
| Correctly refused | 5 of 5 | 3 of 5 |
| Invented an answer | 0 | 2 (the two traps) |
| Citations verified against the passages retrieved | 26 of 26 | 17 of 17 |
| Access probes held | 3 of 3 | 3 of 3 |
| Time questions | not run | 5 of 8 |
| Mean cost per answer | $0.000301 (all 25: $0.0075) | $0 |
| Mean time per answer | 2.31 s (max 5.32 s) | not timed |

On 24 September the app chose the cheapest model for all 25 questions by itself. On 28 September the provider's credit was gone, and the archive answered by exact words only.

The two runs are not like for like. Between them the archive grew from 47 to 203 passages and the transcription moved to the larger Whisper model. The drop from 19 to 7 therefore mixes two causes, and we do not split them.

Without the model, 17 of 17 citations are verified by construction: the archive copies the passage itself, so it cannot cite something that was not retrieved. The cost of search by exact words is that a question worded differently from what was said is refused, which is where the 8 wrong refusals come from. The two traps get the nearest real quote instead of a refusal.

## Questions about time, without the model

| Kind | Result |
|---|---|
| A named year ("in 2015") | passed |
| How it changed over time | passed |
| A period with nothing recorded | passed |
| A newer recording of the same mission is flagged | passed |
| No flag across different subjects | passed |
| "The latest" | failed |
| Before a given year | failed |
| A period chosen by the reader | failed |

The three failures are questions worded differently from the transcript, which search by exact words cannot bridge.

## The failure we show

Question S3: "Is there a cheap way to spot cancer before someone feels ill?" With the model, the archive quoted the speaker's wish for affordable, non-invasive screening (first file, 02:17) and missed the method he built, a urine sample and exosomes, in the next four-minute file. Our automatic check marked the answer correct; its citation was not within 30 seconds of the true line. We read every answer by hand and counted it as a failure. It is the 1 in "19 of 20".

## Citations that play

Every citation marker must open the right file at the right second. On the 47-passage index of 24 September, all 101 markers named a file that exists and a second inside it: none past the end of a file, no passage starting past the end of its file. This was not true before the citation clock was fixed: markers had been built on the clock of the whole talk instead of the clock of the file they name.

## Transcription

| Measure | Value |
|---|---|
| Word error, large-v3, three Gates talks, 11,021 reference words | 5.46 % |
| Per talk: 2015, one speaker | 2.1 % |
| Per talk: 2022, one speaker | 2.6 % |
| Per talk: March 2020 conversation, several speakers | 6.8 % |
| Speed on a CPU container, two full runs | 0.70x then 0.55x real time |
| Audio preparation, 74.7 minutes | 35.8 s |

TED's subtitles are edited for reading, so part of each difference is TED's edit, and the error rates are an upper bound. Container speed changed between days and within a run (the first two files of the 28 September run went at 0.30x), so transcription cost is quoted as a range.

## What this test does not prove

We wrote the questions after reading the transcripts. 203 passages is an easy search; a thousand hours would be about 108,000. Six English TED talks are not Synchrone meetings in French. The real test is a pilot on Synchrone's own recordings.
