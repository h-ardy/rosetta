# Rosetta

Rosetta learns a language from a few hundred translated sentences. It writes its own dictionary and grammar notes, translates new sentences with a word-by-word gloss, and says how sure it is, so a worker knows when to wait for a human interpreter. Every word it learns attaches to a shared map of 244 everyday concepts, so languages that have never shared a dictionary connect through meaning.

Built in one day for the Veritas AI International High School Hackathon 2026, Languages track.

**This is a prototype. It does not replace a human interpreter.**

## Open it

- Notebook: [rosetta.ipynb](rosetta.ipynb), or [open in Colab](https://colab.research.google.com/github/h-ardy/rosetta/blob/main/rosetta.ipynb)
- 3D concept map: https://h-ardy.github.io/rosetta/
- Write-up: see the submission

## Languages tested

| Language | Region | Data | Direction |
| --- | --- | --- | --- |
| Southern Quechua | Andes | AmericasNLP 2021 | Spanish into Quechua |
| Shipibo-Konibo | Peruvian Amazon | AmericasNLP 2021 | Spanish into Shipibo-Konibo |
| Pirahã | Brazilian Amazon | MIT Pirahã Corpus | English into Pirahã |

Each language has 50 test sentences set aside before anything else runs. Results are reported per language, with real outputs shown next to the correct translations.

## How it works

1. **Induction.** A language model (Gemini) reads N sentence pairs and writes a dictionary and grammar notes. Every entry must cite the example pairs it came from, and Python checks every citation.
2. **Translation.** The model translates new sentences using only that dictionary and those notes, and returns a gloss and a list of unknown words.
3. **Confidence.** Plain Python scores each translation from the verified evidence for its words. Below 0.6 it says to wait for an interpreter.
4. **Evaluation.** chrF on the held-out sentences, against a zero-shot baseline and a few-shot baseline (the same examples, no dictionary step). A cipher test swaps every letter of Quechua consistently to check whether Rosetta learns from the examples or recalls the language.
5. **Concept map.** Learned words attach to Concepticon concepts (Swadesh 200 plus Leipzig-Jakarta 100, plus a few intake words). The notebook exports `docs/graph.json` and the site draws it in 3D.

## Run it

Add a Gemini API key in Colab Secrets as `GEMINI_API_KEY`, then Runtime > Run all. Without a key, the notebook replays saved responses from `cache/model_cache.json`.

## Files

| Path | What it is |
| --- | --- |
| `rosetta.ipynb` | The whole pipeline, with outputs |
| `cache/model_cache.json` | Every model response, so the notebook reruns without a key |
| `docs/` | The 3D map site (GitHub Pages) and `graph.json` |
| `outputs/` | Results tables and charts from the run |

## Credits

- [AmericasNLP 2021 shared task data](https://github.com/AmericasNLP/americasnlp2021)
- [MIT Pirahã Corpus](https://osf.io/kt2e8/), Futrell, Stearns, Everett, Piantadosi and Gibson (2016), CC BY-SA
- [Concepticon](https://concepticon.clld.org/) concept lists: Swadesh 1952, Tadmor 2009
- Google Gemini API (free tier)
- pandas, sacrebleu, networkx, matplotlib, scipy, google-genai, [3d-force-graph](https://github.com/vasturiano/3d-force-graph) (MIT license)
- Claude (Anthropic) helped write and debug the code and draft the documentation
