# odin-codex-plugin

**ODIN [for Codex CLI as a plugin]** - **O**utline **D**riven development approach for agentic **IN**telligence

## TL;DR

### Clean Install (Recommended)

```bash
rm -rf ~/.codex/
git clone https://github.com/OutlineDriven/odin-codex-plugin ~/.codex
```

---

### Overwrite Install

```bash
git clone https://github.com/OutlineDriven/odin-codex-plugin
mv ./odin-codex-plugin/* ~/.codex/
```

That easy.

---

## Tips

### Keep upgrade codex up-to-date

```bash
npm i -g @openai/codex@latest
```

- Due to codex’s features are upgraded in silent and sometimes they’re significantly effective, upgrade codex(-cli) on your host machine consistently.

### Set plan mode

- You can turn on plan mode on codex by `shift + tab` shortcut.

### Steering

```bash
 # it trigger `/prompts:askme` prompt, which is designed for gathering more context during planning
askme
```

- Don't need to interrupt by triggering `esc` or wait the plan is fully done. Sometime you feel codex ask you for gathering more context to make better plan.

## Design decisions and codex’s configurations

> Basically, this project already turned on efficient features  of codex, somethings are experimental state but battle-tasted, in .codex/config.toml. This section is going to tell about the decisions.
> 

### [config.toml](./config.toml)

```toml
model_reasoning_effort = "high"
model_verbosity = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
project_doc_fallback_filenames = ["CLAUDE.md"]
model = "gpt-5.4"
personality = "pragmatic"
```

- `model_reasoning_effort`, and `model_verbosity` set high as default due to high performance.
- `project_doc_fallback_filenames` are fallback constitution files. The type of this property is `list[str]`, so you can add `str` type element into this section like `["CLAUDE.md", "AGENTS.md", "CRUSH.md"]`.
    - If you want to reuse other constitutions, feel these props which you’re using before.
- set `personality` as `pragmatic` due to save token(direct communication)
- polished `compaction_prompt` for better context compaction.
- `memories`: codex also supports detecting and saving user’s tastes.

```toml
[agents]
max_depth = 2
max_threads = 96
job_max_runtime_seconds = 36000  # Default: 1800 seconds
```

- codex could call agent(s) in recursively, by following max_depth.
    - But if host machine or your quota is limited, set these values including max_depth appropriately as you want.

### [agents](./agents/)

```toml
model = "gpt-5.3-codex-spark"
```

- some `fast` agents baed on spark model. But this model is only available on ChatGPT pro subscription. So if you're not pro subscriber, change them into appropriate models.

```toml
model = "gpt-5.1-codex-mini"
```

- set `explore` agent use `gpt-5.1-codex-mini`, because they consume little quota buf sufficient for exploration.
