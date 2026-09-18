# Build specifications

A backlog row says *what* to build and how it will be judged. A build specification says *everything a designer and an engineer need to build it* without asking a follow-up question: layout, tokens, every string, every state, the data contract, the keyboard map, the edge cases and the acceptance truths.

One spec exists per screen, written only when that screen is about to be designed.

| Spec | Story | Pipeline stage | Status |
| --- | --- | --- | --- |
| [`TXN-01-pre-submission-gauntlet-release.md`](TXN-01-pre-submission-gauntlet-release.md) | Gauntlet — must resolve, review before release, recorded advisory | Transmission | v1.0 — reference template |

`TXN-01` is the template. Every other spec follows its section order:

1. Story header · 2. The story · 3. Scope · 4. Glossary · 5. Authority matrix · 6. Navigation · 7. Data contract · 8. Layout · 9. Component inventory · 10. Visual specification · 11. Copy deck · 12. Gating rules · 13. Screen states · 14. Interaction flows · 15. Accessibility · 16. Performance · 17. Audit · 18. Edge cases · 19. Acceptance criteria · 20. Test scenarios · 21. Codebase findings · 22. Open decisions · 23. Definition of done · 24. Sources.

Two rules hold across all of them:

- **Every claim is sourced.** A statement about the blueprint cites its section; a statement about the code cites the file and line at a named commit. Nothing is inferred and presented as fact.
- **Acceptance criteria are typed truths**, in the convention of `VAERION_User_Storyv2_1.xlsx`: `[ui]`, `[state]`, `[money]`, `[evidence]`, `[rbac]`, `[event]`, `[a11y]`, `[absence]`. An `[absence]` truth names a state the interface must never render.
