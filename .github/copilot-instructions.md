# Copilot Instructions — Home Assistant Config

This repository contains a Home Assistant configuration written in YAML, using
Home Assistant's Jinja2 template engine.

## Project Structure & Conventions

### Layer responsibilities

| Folder | Purpose |
|---|---|
| `integrations/` | Template sensors and binary sensors — the single source of truth for derived state and alert conditions. Put computed values and thresholds here. |
| `helpers/` | `input_number` / `input_boolean` / `input_datetime` entities for user-configurable values (rates, thresholds, baselines). Never hardcode a rate or threshold in a template — read it from a helper. |
| `automations/` | Reactions only. Automations trigger on binary sensors defined in `integrations/`; they do not re-evaluate conditions inline. |
| `scripts/` | Reusable, parameterized action sequences called by multiple automations. Use `fields:` for inputs. |
| `mqtt/` | MQTT sensor definitions (e.g., water/gas meters from ESPHome). |
| `sensors/` | Statistical and utility sensors that don't fit the template-sensor pattern. |

### Key conventions

- **Binary sensors are the source of truth for alert conditions.** Automations trigger on `binary_sensor.*` state changes; the condition logic lives in `integrations/`, not in the automation.
- **Configurable values live in `helpers/`.** Rates (e.g., `input_number.water_tier1_rate`) and thresholds are set there and read via `states('input_number.*')` in templates. Do not hardcode them.
- **Scripts are parameterized with `fields:`.** When the same action sequence is needed in multiple automations, extract it to `scripts/` and call it with `script.turn_on` + `variables`.
- **One `unique_id` per entity.** Every template sensor and binary sensor must have a `unique_id` so it can be managed via the UI.

## Jinja2 Template Safety Rules

### Always coerce before filtering

Never apply `| round()`, `| int()`, or arithmetic directly to a value that
could be a string, `None`, `unavailable`, or `unknown`. This includes:

- `states('sensor.*')` — always returns a string
- `state_attr('entity', 'attribute')` — returns `None` when the entity or
  attribute is missing

**Always coerce first:**

```yaml
# Bad — crashes if the value is a string or None
{{ state_attr('sensor.foo', 'bar') | round(1) }}
{{ states('sensor.foo') | round(2) }}

# Good
{{ state_attr('sensor.foo', 'bar') | float(0) | round(1) }}
{{ states('sensor.foo') | float(0) | round(2) }}
```

Use `| float(0)` as the default coercion unless the value is explicitly
expected to be an integer, in which case use `| int(0)`.

### Filter precedence in arithmetic expressions

Jinja2 filters bind tightly to the immediately preceding value. When combining
two filtered values with `+`, the final filter applies only to the last term,
**not** to the sum.

**Always parenthesize the addition before applying a filter to the total:**

```yaml
# Bad — | round(2) applies only to sewer_rate, not the sum
${{ water_rate | float(0) + sewer_rate | float(0) | round(2) }}

# Good
${{ (water_rate | float(0) + sewer_rate | float(0)) | round(2) }}
```

### State string guards

When a template branches on a sensor's value, guard against non-numeric states:

```yaml
{% if states('sensor.foo') not in ['unavailable', 'unknown', 'none'] %}
  {{ states('sensor.foo') | float(0) }}
{% endif %}
```
