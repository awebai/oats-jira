# oats-jira

Official [OATS](https://github.com/awebai/oats) tasks-layer integration for Jira. It contributes the `jira-tasks` skill, task-layer instructions, and an advisory spawn hook that gives each instance an `agent-<instance>` label identity and reports the configured Jira site/project.

Jira owns durable task status and outcomes; the selected messaging integration owns conversation. The package never changes Jira assignees, authenticates on an agent's behalf, or makes network calls from lifecycle hooks.

## Requirements

Install Atlassian's `acli` and authenticate it as the human operator:

```bash
acli jira auth status
acli jira auth login --web   # human-run only, when status says unauthorized
```

Installation guide: <https://developer.atlassian.com/cloud/acli/guides/install-acli/>

Requires OATS `>=0.26.0` (the workspace model). The vendored schemas are
described in [`SCHEMA-STATUS.md`](SCHEMA-STATUS.md).

## Declare and select

Declaring the package in the workspace file's `packages:` is the decision to
trust it — its spawn hook runs on every machine that spawns a soul using it —
and `oats sync` locks it to an exact commit and integrity. Nothing is installed.

```yaml
# oats-workspace.yaml
packages:
  oats.jira: v1.0.1
defaults:
  tasks: { oats.jira: { from: package } }   # every soul's tasks slot
```

A soul can instead select it for itself and carry its own settings:

```yaml
# souls/<name>/soul.yaml
capabilities:
  oats.jira: { from: package }              # fills the tasks slot
tasks: { site: example.atlassian.net, project: PROJ }
```

`site` and `project` have three homes: the soul's `tasks:` payload when they
are true of every instance of the soul; the deployment's `oats-local.yaml`
`settings.oats.jira.{site,project}` when they are a fact about this machine;
`oats spawn <soul> --provider oats.jira project=PROJ` for one spawn. Then:

```bash
oats sync --dir <deployment>
oats spawn <soul> --preview    # shows the merged settings.oats.jira
```

Load the `jira-tasks` skill before reading or changing tickets. Its commands and identity/state rules are the package's supported protocol.

## Development

```bash
npm test
```

This validates both manifests, checks resource containment, and smoke-tests the advisory hook.
