# What the platform taught us

[Back to the README](../README.md)

Ten things we hit while building L'archive on Dataiku DSS 15.0.1 (Dataiku Cloud), each found by testing on the instance, and what each one changed.

**1. A failed model call arrives as a success.** The LLM Mesh answers HTTP 200 even when the provider refused the call; the real status is a flag inside each response. When the provider's credit ran out, every call still said 200. Our code reads the per-call flag, so the failure was reported as a failure rather than shown as an empty answer, and a circuit breaker stopped calling a dead model.

**2. The shared code environment installs the wrong "whisper".** It pins `whisper==1.1.10`, which is Graphite's time-series database, not speech recognition. The real speech model lives in two other environments, with `faster-whisper==1.2.1`.

**3. The built-in speech recipe has no timestamps.** Dataiku's Speech Recognition recipe returns the path, the text and a comment, from `.wav` files only. Without the second a sentence was said, an answer cannot be played, so transcription is a Python recipe written by hand.

**4. Vectors written through one handle disappear.** Writing to a knowledge bank through `as_langchain_vectorstore()` looks like it works: the write succeeds and a search in the same run finds the passages. Then the container ends and the vectors are gone, because that handle is a local copy. The next step searched an empty store, keyword search carried the answers, and nothing looked broken. Writes now go through `kb.get_writer()`, whose files upload when the block closes, and the recipe checks hit counts after writing.

**5. The local vector store cannot do hybrid search.** `hybrid_search()` raises `NotImplementedError` on local Chroma. Its metadata filters do work (equal, not equal, ranges, in, and, or), which is what the access rule needs. The keyword search (BM25) is written by hand.

**6. Citations have to be per line.** A passage that crosses a four-minute cut, cited at its start, points into the wrong file for every line after the cut. Each line now carries its own marker on the clock of the file it names.

**7. A fused score cannot decide a refusal.** The rank-fused score is bounded, and in testing it put an unanswerable question above an answerable one. The raw distance in meaning separates them: 0.66 to 1.09 for answerable questions, 1.65 for an unanswerable one.

**8. The webapp cannot query the knowledge bank.** None of the 14 code environments on the instance has both Flask and LangChain. The webapp computes the same vectors itself, and we checked it returns the same top passage on 5 audited questions. At a thousand hours the index must serve the search, which needs one environment an administrator installs.

**9. Container speed varies.** The same transcription ran at 0.70x real time on 17 September and 0.55x on 28 September, with two files at 0.30x. Transcription cost is quoted as a range, never as one number.

**10. The client library cannot create a webapp on this version.** `create_webapp` from `dataikuapi` is rejected by DSS 15.0.1 with "Required field 'name' is missing". The server assigns its own id, and deleting a webapp through the public API answers 405, so a webapp created by mistake cannot be removed. Our deploy script creates once and then overwrites in place.
