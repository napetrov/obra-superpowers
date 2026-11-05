# Harmful spots: obra/superpowers, round 3 (Haiku 5.5 and Sonnet 5.5)


## Result

- Degraded atoms (correct without the skill, wrong with it): **0** for Haiku, **0** for Sonnet, across 478 atoms (dokimi `degraded_lenient` and `degraded_strict` are 0 on every row of `skills.csv`).
- Judge-noise filter: 0 degraded candidates, so **0 dropped**.
- Red spans (kept degraded): 0. Conflicts: not measured (classify step off).
- Blue spans (wrong with and without the skill): **1** candidate, verified below. It is not a skill-caused harm in dokimi's sense, because the no-skill answer is also wrong.

## Verified spot (blue, 1)

| skill | model | file:line | quoted text | answer without skill | answer with skill | model |
|---|---|---|---|---|---|---|
| brainstorming | Haiku 5.5 (judge Sonnet 5.5) | `skills/brainstorming/SKILL.md:164-166` | "When there's genuinely one sensible way, say so in that message and explain why the others don't fit; the design comes in the next one." | "If only one sensible approach exists, you should take it. With no real alternative, there is no decision to deliberate over..." (judge: wrong_other, conf 0.5) | "If there's only one sensible approach, say so plainly and explain why the others don't fit. Then give the short design (...) and ask your partner to approve it. Don't offer a menu of options." (judge: wrong_other, conf 0.9) | us.anthropic.claude-haiku-5-5 |

Atom probed: `afd85710d8f` ("When there is genuinely only one sensible way, the assistant should say so in the approaches message and explain why the others don't fit; the design comes in the next message."). Probe question: "What should be done if only one sensible approach exists?"

Reading:
- Both arms miss the atom. The no-skill answer skips the approaches message altogether. The with-skill answer does say "say so plainly and explain why", but then puts the design in the same reply ("Then give the short design"), which is the opposite of the skill's "the design comes in the next one".
- So the skill sentence is the thing the model misreads. The text is not wrong, but it is easy to misread when the design step follows in the same paragraph. Candidate for a clearer wording, not for removal.
- The no-skill grade is at confidence 0.5, so the judge is unsure. Sonnet judged a Haiku answer; the self-judging caveat applies to the Sonnet arm only.

## Not found

- Sonnet 5.5: no degraded atoms, no blue spans, no red spans.
- Other 14 skills, both models: no degraded atoms and no blue spans. Only `brainstorming` has a blue span.
