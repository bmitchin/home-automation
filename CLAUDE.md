# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant home automation project. HA OS is installed on dedicated hardware.
All configuration is managed as code in this git repository.

**Read `docs/PRINCIPLES.md` before making any configuration changes.** It defines the
authoritative patterns for file organization, naming, secrets management, version control,
and testing.

## Repository Layout

```
configuration.yaml           # HA root config — structural only, no logic here
secrets.yaml                 # NEVER committed — gitignored
secrets.yaml.template        # Committed placeholder listing all required secret keys
packages/                    # Domain-scoped config files (lighting, climate, etc.)
custom_components/           # HACS and manual custom integrations
www/                         # Local Lovelace frontend resources
blueprints/                  # Automation blueprints
docs/
  PRINCIPLES.md              # Guiding principles — the north star for this project
utilities/                   # Tooling and AppImages
```

## Working With This Codebase

### Config Changes

- **New automations** go in `packages/<domain>.yaml`, not `configuration.yaml`
- **All automations must have an explicit `id` field** — treat it as permanent, never change it
- **Secrets** are referenced via `!secret key_name` — never hardcode credentials
- **Entity IDs** follow the pattern `<domain>.<area>_<description>` (lowercase, underscores)
- When adding a new `!secret` reference, also add the key to `secrets.yaml.template` with a `CHANGEME` value

### Restart vs Reload

Before suggesting a restart, check whether a reload is sufficient:

| Change type                     | Use                                               |
|---------------------------------|---------------------------------------------------|
| Automations / Scripts / Scenes  | Developer Tools > YAML > Reload (respective type) |
| Template sensors / Helpers      | Developer Tools > YAML > Reload (respective type) |
| New integration or structural change to `configuration.yaml` | Full restart |

Always run **Developer Tools > YAML > Check Configuration** before a full restart.

### YAML Style

- 2-space indentation
- Block comment at the top of any non-trivial automation explaining its intent
- Comment the "why" not the "what"
- Prefer explicit over implicit (`state: "on"` rather than assuming defaults)

### Version Control

- Commit after every meaningful working change
- Imperative commit messages: "Add sunset lighting automation" not "updated lighting"
- Never commit `secrets.yaml`, `.storage/`, `home-assistant.log`, `*.db`, or `backups/`
- Commit the known-good state before making structural changes

## Lint / Validate

```bash
# Local YAML syntax check (does not understand !secret or !include)
yamllint packages/lighting.yaml

# Full HA config check (requires SSH add-on or HA CLI)
ha core check
```
