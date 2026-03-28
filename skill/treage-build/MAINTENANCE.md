# Treage Skill — Maintenance Guide

## Versioning

The skill version (in `SKILL.md` frontmatter) tracks independently from the engine version.
The skill only needs a version bump when the **generatable surface** changes — meaning something
an LLM needs to know to produce valid CONFIG or TREE objects.

```
Skill v1.1.0 → compatible with Treage engine v1.5.0
```

---

## When to update the skill

### Always update when:
- A new field is added to TREE nodes (e.g., a new optional property)
- A new CONFIG section or key is introduced
- A reserved type name changes or a new reserved type is added
- The CDN version tag in the template and examples needs bumping
- Script load order requirements change

### Update `references/grammar.md`:
- Add new field to the node fields table
- Add new feature to the "Features by version" table
- Update the CDN URL version tag

### Update `assets/boilerplate.html`:
- Bump the CDN script tag to the new version tag
- Add any new required CONFIG fields with sensible defaults

### Update `examples/`:
- Ensure examples still use valid CONFIG/TREE for the new version
- Add a new example if a new feature benefits from a concrete demonstration

### Update `SKILL.md`:
- Add new failure modes if the new feature introduces new ways to break
- Bump `treage-engine` in frontmatter to match

---

## When NOT to update the skill

- Rendering fixes (node layout, spacing, font tweaks)
- UI-only additions (new toolbar button that requires no CONFIG/TREE changes)
- Performance improvements
- Bug fixes that don't affect valid CONFIG/TREE shape

---

## Update checklist (on new Treage release)

1. Read the changelog in the `treage.js` header comment
2. Identify any entries that affect CONFIG or TREE
3. If none: no skill update needed. Done.
4. If yes:
   - [ ] Update `references/grammar.md` — schema tables, features table
   - [ ] Update `assets/boilerplate.html` — CDN tag, any new fields
   - [ ] Update examples if needed
   - [ ] Update `SKILL.md` — failure modes, `treage-engine` frontmatter field
   - [ ] Bump skill version in `SKILL.md` frontmatter (patch for minor additions, minor for structural changes)
   - [ ] Commit with message: `chore: bump treage-skill to v1.x.x for engine v1.x.x`

---

## Changelog

| Skill version | Engine version | Notes |
|---|---|---|
| 1.0.0 | 1.4.0 | Initial release |
| 1.1.0 | 1.5.0 | Workflow shape guidance (Step 2a), schema source note, three new failure modes, open issues section referencing #27 #28 #29 |