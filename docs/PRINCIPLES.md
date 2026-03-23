# Home Automation Project — Guiding Principles

This document is the north star for how this Home Assistant project is structured, maintained,
and extended. It is opinionated by design. Following these principles consistently will keep
the config readable, recoverable, and trustworthy as it grows.

These principles are written for someone learning as they go. They won't all apply on day one,
but they are here so every decision made along the way points in the same direction.

---

## 1. File Organization — Use the Packages Pattern

### The Problem With a Monolithic configuration.yaml

Home Assistant's default is to pile everything into `configuration.yaml`. This works for a
handful of entries but becomes unmanageable fast. Searching for a broken automation in 400
lines of mixed YAML is painful and error-prone.

### The Solution: HA Packages

The **Packages** pattern splits configuration by functional domain. Each file is self-contained
and responsible for one area of your home.

**Enable packages in `configuration.yaml`:**

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

**Target directory layout:**

```
packages/
  lighting.yaml
  climate.yaml
  security.yaml
  media.yaml
  presence.yaml
  notifications.yaml
  utilities.yaml
```

Each package file can contain `automation:`, `script:`, `input_boolean:`, `template:`,
`sensor:`, etc. — all scoped to that domain. Everything related to lighting lives in
`packages/lighting.yaml`. Nothing bleeds across files unless there is a deliberate dependency.

**What stays in `configuration.yaml`:**

- The `homeassistant:` block (name, unit system, time zone, packages include)
- Top-level `http:`, `logger:`, `recorder:`, `history:` config blocks that have no natural
  domain home
- `default_config:` (keep this — it enables many integrations automatically)

Do not put automations, scripts, or helpers directly in `configuration.yaml`.

### Helpers (input_boolean, input_select, counter, etc.)

Helpers created via the UI are stored in `.storage/` and are NOT in version control.
Helpers that are part of the config logic (e.g., `input_boolean.guest_mode`) belong in
the relevant package YAML file so they are explicit, documented, and committed.

---

## 2. Secrets Management — No Hardcoded Credentials

### The Rule

Never put passwords, API keys, tokens, IP addresses of external services, or any sensitive
value directly in a YAML file that is committed to git. Not even once.

### How HA Handles This: secrets.yaml

HA has a built-in mechanism. Create `secrets.yaml` in the config root (same level as
`configuration.yaml`). This file is **never committed** — it is in `.gitignore`.

**secrets.yaml format:**

```yaml
# Home network
wifi_password: your-wifi-password-here

# MQTT broker
mqtt_username: ha_user
mqtt_password: supersecretpassword

# Notification services
pushover_api_key: abc123
pushover_user_key: xyz789
```

**Reference a secret in any config file:**

```yaml
mqtt:
  broker: 192.168.1.10
  username: !secret mqtt_username
  password: !secret mqtt_password
```

### What Is Safe to Commit

- Internal IP addresses of your own devices (192.168.x.x) are borderline — prefer secrets
  for anything that would be useful to an attacker if the repo were made public
- Device names, entity IDs, area names — fine to commit
- Anything that authenticates you to a third-party service — never commit

### secrets.yaml.template

Keep a `secrets.yaml.template` committed to the repo listing all expected keys with
placeholder values. This serves as documentation for what secrets are required without
exposing the real values.

```yaml
# Copy this file to secrets.yaml and fill in real values.
# secrets.yaml is gitignored and must never be committed.

mqtt_username: CHANGEME
mqtt_password: CHANGEME
pushover_api_key: CHANGEME
```

When you add a new `!secret` reference to a config file, add the corresponding key to
`secrets.yaml.template` at the same time.

---

## 3. Naming Conventions

Consistent naming is the single biggest quality-of-life investment in a HA config.
Inconsistent names mean every automation and template needs to be written differently
for every device. Consistent names mean copy-paste works and searches are reliable.

### Entity IDs

HA generates entity IDs from device names. The generated ID is often acceptable but
sometimes needs to be customized. The pattern is:

```
<domain>.<area>_<description>
```

Examples:
- `light.living_room_ceiling`
- `light.living_room_lamp_left`
- `switch.kitchen_coffee_maker`
- `binary_sensor.front_door_contact`
- `climate.bedroom_thermostat`
- `media_player.living_room_tv`

Rules:
- All lowercase, underscores only (no hyphens, no spaces)
- Area prefix first, then the specific device or function
- Descriptive enough that the ID is unambiguous without reading the friendly name

### Friendly Names (UI-facing)

Use Title Case with spaces:
- "Living Room Ceiling Light"
- "Front Door Contact"
- "Bedroom Thermostat"

### Areas

Every device must be assigned to an Area in HA (Settings > Areas & Zones). Use consistent,
short names that map to physical rooms:

```
Living Room, Kitchen, Bedroom, Office, Front Entry, Garage, Backyard
```

Do not create areas for abstract groupings ("Smart Devices", "Lights"). Areas are physical
spaces only. Use Labels for logical groupings.

### Labels

Labels (available in HA 2024.4+) are for cross-cutting classification:

- `critical` — devices whose failure should trigger a notification
- `presence_relevant` — devices used in occupancy detection
- `managed_by_yaml` — helpers defined in YAML (not UI)

### Automation and Script IDs

All automations defined in YAML must have an explicit `id` field:

```yaml
automation:
  - id: lighting_living_room_on_at_sunset
    alias: "Living Room — On at Sunset"
```

The `id` is **permanent and must never be changed** once set. HA uses it to track automation
history, enabled/disabled state, and traces. The `alias` is the human-readable name shown
in the UI and can be changed freely.

ID format: `<package>_<brief_description_in_snake_case>`

---

## 4. UI vs YAML — Know Which Tool to Use

### Use the UI For

- **Integrations**: Always add integrations (Zigbee, Z-Wave, HACS, etc.) through
  Settings > Integrations. Never try to add these in YAML unless the integration
  explicitly requires it.
- **Device discovery**: Let HA discover devices automatically. Rename and assign areas
  in the UI after discovery.
- **Dashboard (Lovelace)**: Build dashboards in the UI editor first. Export to YAML only
  when you need version control on a specific dashboard or need features the UI editor
  doesn't support.
- **Prototyping helpers**: It's fine to create a test `input_boolean` via the UI to
  prototype an automation. Migrate it to YAML once it's permanent.

### Use YAML For

- **All automations**: Every automation kept permanently belongs in YAML. UI-created
  automations are stored in `.storage/automations.json` and cannot be easily reviewed
  or diffed in git.
- **Scripts and Scenes**: Same rule as automations.
- **Template sensors and binary sensors**: Logic should be visible and reviewable in git.
- **Helpers that drive logic**: Any `input_boolean`, `input_select`, `counter`, or `timer`
  that an automation depends on should be in YAML.

### The Migration Path

It is fine to prototype an automation in the UI to get it working. Once it works: open
Settings > Automations > (your automation) > Edit in YAML, copy the YAML into the
appropriate package file, add an explicit `id`, and delete the UI version.

---

## 5. Version Control Workflow

### What to Commit

```
configuration.yaml
secrets.yaml.template        # template only — never secrets.yaml
packages/
custom_components/
www/
blueprints/
themes/
docs/
```

### What to Never Commit (add to .gitignore)

```
secrets.yaml
.storage/
home-assistant.log
home-assistant_v2.db*
backups/
*.tar
```

### Commit Cadence

Commit after every meaningful working change. Do not batch a week of changes into one
commit — that defeats version control. Good timing:

- After adding a new device and getting it working
- After writing or refining an automation
- Before any significant experiment (gives you a known-good state to revert to)

### Commit Message Style

Use the imperative mood: "This commit will..."

```
Add occupancy automation for living room
Fix sunset offset calculation in lighting package
Add secrets.yaml.template with MQTT and Pushover keys
Refactor lighting package — split scenes and automations
```

Avoid vague messages like "update config", "fix stuff", or "wip".

### Branching

For small, low-risk changes: commit directly to main.

For larger experiments (new integration, restructuring packages): use a feature branch.
Merge to main only after the change is tested and working.

```bash
git checkout -b feature/add-zigbee-lighting
# ... work and test ...
git checkout main && git merge feature/add-zigbee-lighting
```

---

## 6. Reliability Practices

### Always Run Config Check Before Restarting

Before every restart, validate the configuration:

- **UI**: Developer Tools > YAML > Check Configuration
- **CLI** (if SSH add-on is installed): `ha core check`

A restart without a config check is a reliability risk. It takes five seconds and
prevents outages.

### Quick Reload vs Full Restart

Many config changes do not require a full restart. Prefer reload over restart:

| Change type                           | Use                                              |
|---------------------------------------|--------------------------------------------------|
| Automations                           | Developer Tools > YAML > Reload Automations      |
| Scripts                               | Developer Tools > YAML > Reload Scripts          |
| Template sensors                      | Developer Tools > YAML > Reload Template Entities|
| Helpers (input_boolean, etc.)         | Developer Tools > YAML > Reload Helpers          |
| Integrations / new device added       | Full restart or integration reload               |
| `configuration.yaml` structural change| Full restart                                     |

Full restarts briefly interrupt all automations. Use them only when required.

### Backups Before Big Changes

Before any significant restructuring or HA version upgrade:
1. Take a full backup: Settings > System > Backups > Create Backup
2. Commit your current state to git
3. Make the change, verify everything works
4. Commit the result

### Keep HA Updated — But Not Immediately

HA releases monthly. Wait 1–2 weeks after a major release before updating. Read the
release notes and **Breaking Changes** section before every update. Never update when
you need the system to be stable.

---

## 7. Documentation in Config — Comment Your YAML

### Comment the "Why", Not the "What"

The YAML already says what it does. Comments should explain reasoning.

**Too obvious:**
```yaml
# Turn on the light
service: light.turn_on
```

**Useful:**
```yaml
# Offset by -20 minutes because the sensor is in a west-facing window
# and triggers ~20 minutes after actual sunset at this latitude
trigger:
  - platform: sun
    event: sunset
    offset: "-00:20:00"
```

### Block Comments on Complex Automations

Open complex automations with a comment explaining the overall intent:

```yaml
# AUTOMATION: Evening Wind-Down (Weeknights)
# Triggered at 9pm Mon–Fri. Dims all lights to 30%, sets thermostat
# to 68F, and enables the do-not-disturb scene.
# Weekend equivalent: weekend_wind_down (packages/climate.yaml)
- id: climate_evening_wind_down_weeknights
  alias: "Evening Wind-Down (Weeknights)"
```

### Document Cross-Package Dependencies

If a package uses a helper or entity defined in a different package, say so:

```yaml
# Requires input_boolean.guest_mode (defined in packages/presence.yaml)
condition:
  - condition: state
    entity_id: input_boolean.guest_mode
    state: "off"
```

---

## 8. Testing Approach

### Test Jinja2: Developer Tools > Template

Before putting a template in a config file, test it in Developer Tools > Template.
This is the fastest feedback loop available. Paste your template, see the output
immediately. Use this for `value_template:`, condition templates, and notification bodies.

**Workflow:** Write template → verify output → paste into config file. Never skip step 1.

### Test Actions: Developer Tools > Actions

Use Developer Tools > Actions to call any service manually without waiting for a trigger
to fire. This lets you test the `action:` block of an automation in isolation.

### Trigger Automations Manually

In Settings > Automations, every automation has a "Run" button. Use this to test the
full automation (conditions + actions) without waiting for the real trigger.

### Read the Automation Trace

After an automation runs, navigate to Settings > Automations > (your automation) > Traces.
The trace shows exactly which trigger fired, which conditions passed or failed, and which
actions executed. Check the trace before editing a "broken" automation — the problem is
usually visible immediately.

### Validate YAML Syntax Locally

Run the HA config check before every restart (Developer Tools > YAML > Check Configuration).
For local pre-commit validation, `yamllint` catches syntax errors without needing HA running:

```bash
yamllint packages/lighting.yaml
```

Note: `yamllint` does not understand HA-specific constructs (`!secret`, `!include`) but
catches indentation errors and syntax problems before you push.

---

## Quick Reference

### New Device Checklist
1. Add integration via UI (Settings > Integrations)
2. Rename entity to follow naming convention
3. Assign to correct Area, add relevant Labels
4. If device needs custom logic, update the relevant package file
5. Run config check → commit

### New Automation Checklist
1. Prototype in UI if needed, then migrate to YAML
2. Write in the appropriate `packages/<domain>.yaml` file
3. Add explicit `id` field and a block comment describing intent
4. Test via "Run" button and inspect the Trace
5. Reload automations (no restart needed) → commit

### Emergency Recovery

If HA fails to start after a config change:
1. Check the logs (Settings > System > Logs, or `/config/home-assistant.log`)
2. Find the last working state: `git diff HEAD~1 HEAD -- packages/affected_file.yaml`
3. Fix the error or revert the file
4. Run config check again before retrying
