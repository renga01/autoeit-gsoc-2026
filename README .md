# AutoEIT — GSoC 2026 Test II
**Automated Scoring System for Spanish EIT Responses**

Renu Kulkarni · rengasings@gmail.com · [github.com/renga01](https://github.com/renga01)

---

## Setup & Usage

```bash
python3 -m venv myenv
source myenv/bin/activate
pip install openpyxl
python eit_scorer.py
```

Input: Place your Excel file in the same folder as the script and update the `INPUT` path at the bottom of `eit_scorer.py`:

```python
# Edit these two lines at the bottom of eit_scorer.py
INPUT  = "/home/your-username/Downloads/Example_EIT Transcription and Scoring Sheet.xlsx"
OUTPUT = "/home/your-username/Downloads/autoeit-gsoc-2026/AutoEIT_Scored_Output.xlsx"
```

> **Tip:** Drag and drop the Excel file into your terminal to get its exact path automatically.
Output: `AutoEIT_Scored_Output.xlsx` — original sheets + `Auto Score` + `Auto Rationale` columns + summary sheet

---

## Approach

The script applies the Ortega (2000) meaning-based rubric in four steps:

**1. Clean the text**
Strip transcription artefacts — false starts `[el li-]`, unintelligible segments `XXX`, filler markers — so they don't affect scoring.

**2. Detect score-0 cases**
Silence `[...]`, English-only responses `[en inglés]`, and responses with only one meaningful word are immediately scored 0.

**3. Compute content word overlap**
Function words (*el, la, de, que...*) are ignored. The script extracts content words (nouns, verbs, adjectives) from both the prompt and the learner's response and calculates what fraction of the prompt's meaning was reproduced.

**4. Check for meaning changes**
Even with high word overlap, three things can lower the score:
- **Antonym substitution** — e.g. *fácil* instead of *difícil* → cap at 2
- **Negation change** — *no* added or dropped → cap at 2
- **Grammar structure loss** — e.g. *al que* simplified to *que* → cap at 3

**Score mapping:**

| Condition | Score |
|-----------|-------|
| ≥ 90% overlap, no meaning/grammar changes | 4 |
| ≥ 65% overlap, meaning preserved | 3 |
| ≥ 35% overlap | 2 |
| 15–35% overlap | 1 |
| < 15% or ≤ 1 content word | 0 |

---

## Results

Validated against 1,560 human-scored sentences across 29 participants:

| Metric | Result |
|--------|--------|
| Exact agreement | 79.9% |
| Within ±1 agreement | 98.0% |
| Weighted Cohen's Kappa | 0.781 |

The 98% within-±1 agreement confirms the system correctly understands the rubric's direction. The gap in exact agreement is mainly due to two things a text-only approach cannot catch: **phonetic errors** (e.g. *preseguido* vs *perseguido* — same content words, but a rater hears the mispronunciation) and **valid synonym substitutions** (e.g. *gente* for *personas*). Both are scoped improvements for the GSoC project — adding fuzzy phonetic matching and a Spanish synonym layer would push exact agreement above 90%.

---

## Dependencies

`openpyxl` only. No ML libraries required — fully reproducible on any Python 3.8+ environment.
