# Sub profiles

Some profiles might have multiple profiles. This is useful when a device has different power consumption based on the state of the device.
This can be used for example for different infrared modes of a light. Or lights having different effects.

Each sub profile has it's own subdirectory `{manufacturer}/{modelid}/{subprofile}`, which contains a `model.json` file.
You can also define `sub_profile_select` in the main `model.json` to automatically select the sub profile based on the state of the device.
When no `sub_profile_select` is defined, the user will be asked to select the sub profile during discovery or while setting up manually from library.

Examples:

- [lifx/LIFX A19 Night Vision](https://github.com/bramstroker/homeassistant-powercalc/tree/master/profile_library/lifx/LIFX%20A19%20Night%20Vision)
- [eufy/T8400](https://github.com/bramstroker/homeassistant-powercalc/tree/master/profile_library/eufy/T8400)

## Naming

Use a stable scheme for profiles that describe feature combinations:

- Prefix every enabled feature with `+`, including the first one: `+wlan_+eco_+bt`.
- List only the features that are enabled. A missing `+led` already means the LED is off.
- Keep one fixed order across the whole model family, for example
  `wlan` → `eco` → `ap` → `bt` → `led`.
- Reserve `-` for legacy exceptions where an explicit off-state is required
  (for example `+wlan_+eco_-led`).

The directory name is what the sub profile picker sorts by, so a consistent order keeps
the selection list readable. The `name` field in the sub profile `model.json` is the label
shown to the user; keep it in the same order as the directory name.

## Legacy ids

Sub profile directories are referenced by their exact name in YAML and in config entries.
Renaming one therefore breaks existing installations. To rename a published sub profile,
add the old directory name to `legacy_ids` in the **new** directory:

```json
{
    "name": "WLAN + ECO",
    "legacy_ids": ["wlan_+eco"],
    "standby_power": 0.3
}
```

Powercalc resolves a legacy id to the directory declaring it, both when loading YAML
and when selecting a sub profile dynamically. Config entries are rewritten to the
canonical name during setup. Exact directory names always win over a legacy id, so an
old name can never shadow a real directory.

When the parent model uses `sub_profile_select`, matcher values and the `default` must
point to the canonical directory name. Legacy ids are accepted as well, but keeping the
matchers canonical avoids a second lookup.

## Sub profile select

### Entity ID

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "entity_id",
                "pattern": ".*_nightlight$",
                "profile": "nightlight"
            }
        ],
        "default": "default"
    }
}
```

## Entity state

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "entity_state",
                "entity_id": "select.{{source_object_id}}_infrared_brightness",
                "map": {
                    "Disabled": "infrared_off",
                    "25%": "infrared_25",
                    "50%": "infrared_50",
                    "100%": "infrared_100"
                }
            }
        ],
        "default": "infrared_off"
    }
}
```

## Attribute

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "attribute",
                "entity_id": "select.{{source_object_id}}_infrared_brightness",
                "map": {
                    "effect1": "effect1",
                    "effect2": "effect2"
                }
            }
        ],
        "default": "default"
    }
}
```

## Integration

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "integration",
                "integration": "deconz",
                "profile": "deconz"
            }
        ],
        "default": "default"
    }
}
```

## Entity registry property

Use this when a sub profile depends on any field from the Home Assistant entity registry entry,
for example `translation_key`, `device_class`, `entity_category`, `original_name`, or `aliases`.

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "entity_registry",
                "property": "translation_key",
                "value": "smart_plug",
                "profile": "smart_plug"
            }
        ],
        "default": "default"
    }
}
```

## Model ID

Could be used when a single physical device is represented by multiple devices in Home Assistant.
For example Hue Datura light has a top and bottom light, these are 2 different devices in HA.

See: [signify/929003736201](https://github.com/bramstroker/homeassistant-powercalc/tree/master/profile_library/signify/929003736201)

```json
{
    "sub_profile_select": {
        "matchers": [
            {
                "type": "model_id",
                "model_id": "929003736201_02",
                "profile": "bottom"
            }
        ],
        "default": "top"
    }
}
```
