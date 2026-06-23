# /transcript-init

Bootstrap the vault for a new project. Run once to set the project name and stakeholder roster.

---

## Steps

### 1. Check existing configuration

Read `vault/roster.yaml`.

If the file contains a non-empty `people:` list (one or more entries), warn:

> This vault already has [N] stakeholders configured. Re-running init will overwrite `vault/roster.yaml`. Continue? (yes/no)

If the user says no, stop immediately.
If the file does not exist, or `people:` is empty (`[]`), proceed without prompting.

---

### 2. Collect project name

Ask: **"What is the project name?"**

---

### 3. Collect stakeholders (conversational loop)

Say: "Let's add stakeholders one at a time."

For each stakeholder, ask in sequence:
1. **"Stakeholder name?"**
2. **"Role (e.g., CFO, PM, Engineer)?"**
3. **"Domains — what topics does this person own? Enter as a comma-separated list (e.g., finance, cost, budget)"**
4. **"Aliases — any shorthand names or initials? (comma-separated, or press Enter to skip)"**

After each entry, ask: **"Add another stakeholder? (yes/no)"**

Stop when user says no or any negative.

---

### 4. Write `vault/roster.yaml`

Write with this exact structure:

```yaml
project_name: <project_name>
people:
  - name: <name>
    role: <role>
    domains: [<domain1>, <domain2>]
    aliases: [<alias1>, <alias2>]
```

If a stakeholder provided no aliases, write `aliases: []`.

If the user added no stakeholders:

```yaml
project_name: <project_name>
people: []
```

---

### 5. Confirm

Print:

> Vault configured: **<project_name>**
> Stakeholders added: <name1> (<role1>), <name2> (<role2>), ...

If no stakeholders were added, print instead:

> Note: No stakeholders configured. All speakers will receive `authority: unknown` during scan.

Always append:

> To add stakeholders later, edit `vault/roster.yaml` directly.
