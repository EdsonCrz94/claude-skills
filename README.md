# Claude Skills

Agent Skills I use for my own work, published so others can reuse them.

A skill is a folder with a `SKILL.md` file: YAML frontmatter that tells Claude when the skill applies, followed by Markdown instructions Claude loads only when it needs them. Skills follow the [Agent Skills](https://agentskills.io) open standard, so they work across tools that implement it.

## Skills

| Skill | What it does |
|---|---|
| [`power-apps-sharepoint`](power-apps-sharepoint/) | Conventions for building Power Apps canvas apps on SharePoint / Microsoft Lists: control naming, modern-control policy, reusable top bar and sidebar, list and column design, indexes, foreign keys, a mandatory admin configuration module, delegation rules, and known platform traps. |

### power-apps-sharepoint

Written from real production work on canvas apps backed by SharePoint lists. It is prescriptive rather than exploratory: every rule is either a convention that keeps a growing app consistent, or a trap that already cost rework once.

What it covers:

- **Naming.** A prefix table for every control type, plus patterns for screens, components, variables, collections and flows.
- **Language split.** Technical names in English, end-user text in Spanish.
- **Controls.** Modern controls by default, with explicit criteria for when a classic control is justified.
- **UI.** Minimalist layout with a reusable top bar and sidebar, and a single theme record so no control holds a literal color.
- **Data model.** List and column naming, mandatory column descriptions, layer separation (security, catalog, configuration, transactional, log), capability flags for permissions, and snapshotting reference values into transactional rows.
- **Configuration module (mandatory).** Every app gets admin screens and a settings list for its business parameters (intervals, thresholds, recipients, switches), so a reminder "every 15 days" can become 7 or 30 without editing the app or its flows. Includes the list schema, access control, the edit-screen formulas, and the daily-flow pattern for schedules that must follow a setting.
- **Indexes and foreign keys.** Index every column used to filter, sort or search. Relate lists with an indexed `<Parent>ID` Number column by default; Lookup columns as the documented alternative.
- **Delegation.** A quick-reference table of what does and does not delegate against SharePoint, preferred delegable patterns, and a required reporting format for any warning that cannot be avoided.
- **Platform traps.** Choice column read/write asymmetry, gallery scroll persistence, `TemplateSize` limits, attachments outside an `EditForm`, REST provisioning quirks, and more.

All examples use generic placeholders (`<SiteUrl>`, `<AppCode>`, `<ListName>`, `<ColumnName>`). No tenant, site, list, user or business data is included.

## How to use a skill

**Claude Code — personal (all your projects on this machine):**

```bash
git clone https://github.com/EdsonCrz94/claude-skills.git
cp -r claude-skills/power-apps-sharepoint ~/.claude/skills/
```

**Claude Code — project (shared with your team through the repo):**

```bash
cp -r claude-skills/power-apps-sharepoint <your-project>/.claude/skills/
```

Commit `.claude/skills/` so everyone working in the repository gets it.

**claude.ai:** upload the skill folder from your skills settings.

Once installed, Claude loads the skill on its own when the conversation matches its description, or you can invoke it directly with `/power-apps-sharepoint`.

To confirm it is available, ask Claude: *What skills are available?*

## Adapting a skill

These skills encode the conventions of one team. Fork the repository and edit `SKILL.md` if yours differ. The parts most likely to need changing:

- The language split, if your users do not read Spanish.
- The list naming pattern `<AppCode>_<Module>_<Entity>`.
- The PascalCase column convention.
- The choice of `<Parent>ID` Number columns over Lookup columns.

## Contributing

Issues and pull requests are welcome, especially corrections. If a rule here is wrong or has been superseded by a platform change, open an issue with a reproducible case and I will update it.

## License

MIT — see [LICENSE](LICENSE). Use it, change it, ship it commercially. Keep the copyright notice, and it comes with no warranty.
