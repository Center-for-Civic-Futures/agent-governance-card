# How to read the schema

All field content — questions, enum values, glosses, help lists, derivation
rules — lives in the **canonical Agent Governance Card schema**, which the
skill reads at run time. Resolve it in this order; at the first hit, stop:

1. **Canonical file in a repo checkout** — running inside a clone of the
   project (e.g. Claude Code in the repo). Search upward from this directory
   for `schema/agent-governance-card.schema.yaml`; in the project repo's
   layout that is `../../../schema/agent-governance-card.schema.yaml`. This is
   the freshest copy.
2. **Environment-provided copy** — running where the environment supplies
   documents (a Project's knowledge files, a GitHub connector, an attached
   file): look for a YAML document whose top-level key is
   `schema: agent-governance-card`. An uploaded copy is a *static snapshot* —
   trust its `updated` date, not the calendar.
3. **The copy bundled with this skill** — nothing else found:
   [`schema.snapshot.yaml`](schema.snapshot.yaml) in this directory. When the
   skill is installed on its own (e.g. via `npx skills add`), this is the only
   copy and it is authoritative.

Read the schema silently: do not tell the team which copy, version or date
you loaded.

## Contract check (do this every run)

Read the schema's `derivation_contract_version` and compare it with
`EXPECTED_CONTRACT_VERSION` in [`../scripts/derive_impact.py`](../scripts/derive_impact.py):

- **Equal** → proceed normally.
- **Schema higher** → the derivation rules have changed since this skill was
  built. **Do not compute an Impact Level by any method** (engine or prose) —
  a wrong tier is the worst possible failure. Collect answers if useful, and
  tell the team the skill's engine must be updated to the new contract first.
- **Schema lower** (unusual — an old schema copy) → say the copy predates the
  engine and ask for a fresher schema before deriving.

The split exists so the **content layer** (question wording, glosses, help
text, ordering) can be edited freely and flow into the next run, while the
**derivation contract** (ids + enum values of 2.5, 3.1, 4.1, 4.2, 4.3, 4.4 and
the rules) only changes with an explicit version bump the engine can detect.
Field ids are the document's numbers, as quoted strings (`"2.5"`). A fill keyed
on any other ids is refused by the contract check, not silently mis-derived.
