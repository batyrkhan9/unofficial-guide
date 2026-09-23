# The Unofficial Guide 

This project builds an unofficial guide to Minerva University professors using student reviews collected from Rate My Professors. The domain covers 14 professors across Computer Science, Social Sciences, Arts and Humanities, and Business. This knowledge is valuable because it gives students honest insight into teaching style, exam difficulty, and grading fairness, information that is never shared through official Minerva channels because it is too candid and informal to appear in any university publication.
---

## Document Sources

| # | Source | Type | URL or file path |
|---|--------|------|-----------------|
| 1 | Rate My Professors| Prof Subasic | https://www.ratemyprofessors.com/professor/3158936 |
| 2 | Rate My Professors| Prof Digby | https://www.ratemyprofessors.com/professor/2965507 |
| 3 | Rate My Professors| Prof Gale | https://www.ratemyprofessors.com/professor/2965451 |
| 4 | Rate My Professors| Prof Morar | https://www.ratemyprofessors.com/professor/2977249 |
| 5 | Rate My Professors| Prof Perry | https://www.ratemyprofessors.com/professor/2965452 |
| 6 | Rate My Professors| Prof Powers| https://www.ratemyprofessors.com/professor/2983398 |
| 7 | Rate My Professors| Prof Rios | https://www.ratemyprofessors.com/professor/3075440 |
| 8 | Rate My Professors| Prof Sealfon | https://www.ratemyprofessors.com/professor/3106280 |
| 9 | Rate My Professors| Prof Terrana | https://www.ratemyprofessors.com/professor/2962445 |
| 10 | Rate My Professors| Prof Volkan | https://www.ratemyprofessors.com/professor/2977238 |
| 11 | Rate My Professors| Prof Doering | https://www.ratemyprofessors.com/professor/2977244 |
| 12 | Rate My Professors| Prof Bentsen | https://www.ratemyprofessors.com/professor/2983399 |
| 13 | Rate My Professors| Prof Singh | https://www.ratemyprofessors.com/professor/2962474 |
| 14 | Rate My Professors| Prof Lawry | https://www.ratemyprofessors.com/professor/2965450 |


---

## Chunking Strategy

**Chunk size:**
Each review is one chunk, approximately 50-150 characters. Split on --- separator.
**Overlap:**
No overlap. Reviews are independent opinions with no contextual relationship between them.
**Why these choices fit your documents:**
Reviews average 2 sentences and useful information is spread across both sentences. Splitting smaller would destroy meaning. Since each review stands alone, overlap would add noise rather than context.
**Final chunk count:**
40 chunks across 14 documents.
---

## Embedding Model

**Model used:**
all-MiniLM-L6-v2 via sentence-transformers. Runs locally with no API key or rate limits.

**Production tradeoff reflection:**
For a production deployment, I would consider OpenAI's text-embedding-ada-002 for higher accuracy at a per-token cost. Since Minerva is an international university with students from many countries, multilingual support would also be important, a model like multilingual-e5-large would handle non-English reviews better. The main tradeoff is cost and latency versus accuracy. all-MiniLM-L6-v2 is fast and free but was trained on general text, not student reviews specifically, which may reduce accuracy on domain-specific language.
---

## Grounded Generation

**System prompt grounding instruction:**
You are an assistant for Minerva University students.
Answer the question using ONLY the information provided in the documents below.
If the documents don't contain enough information to answer, say 'I don't have enough information on that.'
Always be specific and cite which professor you are referring to.
**How source attribution is surfaced in the response:**
Source filenames are collected from ChromaDB metadata for each retrieved chunk and returned alongside the answer. The Gradio interface displays them in a separate "Retrieved from" field.

---

## Evaluation Report

| # | Question | Expected answer | System response (summarized) | Retrieval quality | Response accuracy |
|---|----------|-----------------|------------------------------|-------------------|-------------------|
| 1 | Which professor is considered the hardest at Minerva? | Either Prof Subasic or Terrana |Found Terrana but hedged, said couldn't confirm definitively|Partially relevant | Partially accurate |
| 2 | Which professor gives the most useful feedback? | Prof Rios | Couldn't identify professor by name, quoted review without attribution | Off-target | Inaccurate |
| 3 | Who is the harshest grader at Minerva? | Either Prof Subasic or Terrana | Correctly identified Terrana | Relevant | Accurate |
| 4 | Which professor is the friendliest? | Prof Perry or Rios | Found friendly descriptions but couldn't name the professor | Partially relevant | Partially Accurate |
| 5 | What do students say about exam difficulty at Minerva? | Not possible to understand from prof reviews, but not easy exams | Returned no relevant information, said insufficient data | Off-target | Accurate |

**Retrieval quality:** Relevant / Partially relevant / Off-target  
**Response accuracy:** Accurate / Partially accurate / Inaccurate

---

## Failure Case Analysis

**Question that failed:**
Which professor gives the most useful feedback?
**What the system returned:**
The system found a review mentioning "gives really helpful feedback" but could not identify which professor it referred to, returning a vague answer without a name.
**Root cause (tied to a specific pipeline stage):**
The failure occurred at the chunking stage. Original chunks contained only review text without professor names embedded in the chunk itself. The professor name only existed in the filename metadata, which the LLM cannot see directly, it only receives the chunk text. This meant retrieved chunks had relevant content but no attribution. After adding "Professor: [name]" to each review, results improved for some questions but not all, because some reviews still used indirect references like "this prof" rather than explicit names.
**What you would change to fix it:**
Ensure every chunk explicitly contains the professor's name in the text. Additionally, use a larger k value to retrieve more chunks, increasing the chance that at least one chunk contains both the professor's name and the relevant information.

---

## Spec Reflection

**One way the spec helped you during implementation:**
Writing the chunking strategy in planning.md before coding forced me to think about the structure of my documents first. Because I had decided that each review should be one chunk split on ---, implementing the ingestion script was straightforward, I knew exactly what the output should look like before writing a single line of code.

**One way your implementation diverged from the spec, and why:**
The spec did not anticipate that professor names would be missing from chunk text, only present in filenames. This caused retrieval failures where the system found relevant content but couldn't attribute it to a specific professor. I had to go back and modify all 14 .txt files to prepend "Professor: [name]" to each review, a preprocessing step not in the original plan.

