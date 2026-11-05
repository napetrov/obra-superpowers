# Knowledge map data for obra/superpowers

Data behind the knowledge-map issue on obra/superpowers. Made with [dokimi](https://github.com/intel/dokimi) `novelty atoms` + `novelty probe`, 2026-11-04 checkout, Haiku 5.5 and Sonnet 5.5 on Amazon Bedrock. Judge: Sonnet 5.5 (it grades its own answers on the Sonnet rows).

## Files

- `skills.csv`: one row per skill x model (30 rows, 15 skills).
  - `n_paired` is the atoms both arms answered; `n_unpaired_dropped` counts atoms with an `error` verdict.
  - `known_*` is the share correct WITHOUT the skill (dokimi k0), `new_*` the share correct only WITH it (added), `degraded_*` correct without and wrong with (flips), `still_missing_lenient` wrong in both arms.
  - `ci95_new_lenient_*` is dokimi's interval on the new share. At n=25 it is about +/-0.2.
  - `dead_token_share` is the share of SKILL.md tokens in spans that no extracted atom cites.
  - `judge` is always Sonnet 5.5. `self_judged=yes` marks the Sonnet rows.
- `facts.jsonl`: one row per atom per skill (478 atoms). Fields: skill, atom_id, atom_text, kind, tags, span_ids, lines (file:line of each span in `skills/<skill>/SKILL.md`), then `haiku` and `sonnet` blocks with `known_lenient`, `outcome_lenient` and `outcome_strict` (added / redundant / still_missing / degraded / unpaired), `probe_question`, and the no-artifact and full answers with verdict and confidence (answers clipped to 500 chars).
- `harmful.md`: the verified spots (1 blue span, 0 degraded).

## Verdict scale and outcomes

Verdicts from dokimi's judge: `correct_complete`, `correct_missing_qualifier`, `partially_correct` (lenient-correct), `wrong_other`, `wrong_reversed`, `abstained`, `error`. Strict = `correct_complete` only. `abstained` counts as not correct. Only `error` drops an atom from the pair.

Outcome per atom (no-skill arm = base, with-skill arm = full): added (base not correct, full correct), redundant (both correct), degraded (base correct, full not), still_missing (neither). `unpaired` only when one arm errored.

## Charts

`a_known_without_skill.png`: per skill, share of facts each model got right without the skill. `b_what_each_skill_adds.png`: the same facts split into known to both, known to one model, new to both, and hurts.

**Excerpts**: each SKILL.md line takes the colour of the facts drawn from it: dark grey = known to both models without the skill, light grey = known to one model only (mostly noise at 25 facts), green = new to both, red = right without the skill and wrong with it. Uncoloured lines had no probed fact. The pill is the number of facts on the line.

## How to reproduce

Per skill, with `ART="skill:<checkout>/<path to the skill folder>"`:

```
# 1. one atom set (Sonnet 5.5 extracts), 25 atoms max, $0.30 cap
dokimi novelty atoms --artifact "$ART" --max-atoms 25 --no-verify \
  --model us.anthropic.claude-sonnet-5-5 --extract-model us.anthropic.claude-sonnet-5-5 \
  --provider amazon-bedrock --max-usd 0.3 --out out/<skill>/atoms.json

# 2. Haiku 5.5 answers the same atoms, Sonnet 5.5 judges, no paraphrase, no reversal, $0.50 cap
dokimi novelty probe --artifact "$ART" --atoms out/<skill>/atoms.json \
  --model us.anthropic.claude-haiku-5-5 --provider amazon-bedrock \
  --judge-model us.anthropic.claude-sonnet-5-5 --judge-provider amazon-bedrock \
  --no-paraphrase --no-reversal --max-usd 0.5 --out out/<skill>/probes_haiku.json

# 3. Sonnet 5.5 answers the same atoms, Sonnet 5.5 judges (self-judged: needs --allow-self-judge)
dokimi novelty probe --artifact "$ART" --atoms out/<skill>/atoms.json \
  --model us.anthropic.claude-sonnet-5-5 --provider amazon-bedrock \
  --judge-model us.anthropic.claude-sonnet-5-5 --judge-provider amazon-bedrock \
  --allow-self-judge --no-paraphrase --no-reversal --max-usd 0.5 --out out/<skill>/probes_sonnet.json
```
