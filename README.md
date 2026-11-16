# Rosetta

Rosetta learns a language from a few hundred translated sentences. It writes its own dictionary and grammar notes, translates new sentences with a word-by-word gloss, and says how sure it is, so a worker knows when to wait for a human interpreter. Every word it learns attaches to a shared map of 244 everyday concepts, so languages that have never shared a dictionary connect through meaning. A second map connects them through grammar: the suffixes and short words that do the same job in each language.

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
6. **Grammar map.** Suffixes and function words from the dictionaries are grouped by the job they do (object, topic, plural, firsthand knowledge and so on), using Leipzig glossing labels. Python checks that the examples the model cited really contain each suffix.
7. **Community.** Speakers can suggest a word for any concept from the site. Each suggestion opens a public GitHub issue, people vote with reactions, and words labeled `approved` appear on the map.

## Results (first run)

chrF on 50 unseen test sentences per language, from 200 examples. Higher is better.

| Language | Zero-shot | Few-shot | Rosetta | Model |
| --- | --- | --- | --- | --- |
| Southern Quechua | 43.3 | 52.1 | 20.6 | gemini-3.6-flash |
| Shipibo-Konibo | 16.2 | 38.8 | 16.9 | gemini-3.1-flash-lite |
| Pirahã | 15.8 | 40.7 | 22.7 | gemini-3.5-flash-lite |

The free tier allows about 20 requests per model per day, so each language ran on a different model. Compare within a row, not across rows.

Translating only through the learned dictionary loses to pasting the examples in, because Rosetta leaves words it has no evidence for untranslated. The cipher test scored 11.4 against 20.6 on real Quechua, so part of the Quechua score comes from what the model already knew. The confidence flag cleared 4 of 150 translations and sent the rest to an interpreter.

## Results (second run: dictionary plus examples)

The second-run cell gives the translator the 200 examples as well as the dictionary, runs every language on gemini-3.6-flash, and reruns few-shot on that model for a fair comparison. It uses `data/dictionaries_200.json`, the dictionaries from the first run.

| Language | Few-shot | Rosetta + examples |
| --- | --- | --- |
| Southern Quechua | 51.3 | 48.2 |
| Shipibo-Konibo | 44.8 | 44.5 |
| Pirahã | 42.2 | 42.2 |

Rosetta matches the plain prompt within a point on two languages and trails by 3.1 on Quechua, and every translation comes with a word-by-word gloss and a confidence score. Inside each language, sentences the flag marked "use" scored higher than those marked "wait for interpreter" (Quechua 53.1 vs 48.0, Shipibo-Konibo 49.6 vs 43.1, Pirahã 40.2 vs 36.1). Pooled across languages the gap disappears, so the cutoff needs setting per language.

Pirahã dictionary check: 83 of the 111 learned words appear in the corpus word list, and for 55 of them (66%) the learned English meaning shares a word with the linguists' gloss.

## Run it

Add a Gemini API key in Colab Secrets as `GEMINI_API_KEY`, then Runtime > Run all. Without a key, the notebook replays saved responses from `cache/model_cache.json`.

## Files

| Path | What it is |
| --- | --- |
| `rosetta.ipynb` | The whole pipeline, with outputs |
| `cache/model_cache.json` | Every model response, so the notebook reruns without a key |
| `docs/` | The 3D map site (GitHub Pages) and `graph.json` |
| `data/dictionaries_200.json` | The dictionaries and grammar notes the model wrote from 200 examples |
| `outputs/` | Results tables and charts from the run |

## Credits

- [AmericasNLP 2021 shared task data](https://github.com/AmericasNLP/americasnlp2021)
- [MIT Pirahã Corpus](https://osf.io/kt2e8/), Futrell, Stearns, Everett, Piantadosi and Gibson (2016), CC BY-SA
- [Concepticon](https://concepticon.clld.org/) concept lists: Swadesh 1952, Tadmor 2009
- Grammar facts on the site: [WALS](https://wals.info/) feature 81A; Faller 2002 (Cuzco Quechua evidentials); Valenzuela 2003 (Shipibo-Konibo); Everett 1986 (Pirahã)
- Google Gemini API (free tier)
- pandas, sacrebleu, networkx, matplotlib, scipy, google-genai, [3d-force-graph](https://github.com/vasturiano/3d-force-graph) (MIT license)
- Claude (Anthropic) helped write and debug the code and draft the documentation
