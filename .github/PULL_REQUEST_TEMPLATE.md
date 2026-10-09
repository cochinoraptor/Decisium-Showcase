## What

(describe the change in one or two sentences)

## Why

(what problem or opportunity this addresses)

## Scope

- [ ] Documentation only
- [ ] Code: comments / user-facing strings (English)
- [ ] Code: behavior change (with tests)
- [ ] Security-relevant change

## Checks

- [ ] `npm test` in `firewall/` passes (192/192)
- [ ] `npm run build` in `firewall/` exits 0
- [ ] Docs updated where relevant (README, SECURITY_NOTES, DECISIONS, reports)
- [ ] No attack payloads modified; tests still reproduce the same cases

## Notes

- Keep user-facing text and comments in English; do not rename identifiers or config values to translate.
- Security issues: open a PR only after the private disclosure path has been followed, if applicable.
