# Vacuum-Voice-View-Assist
View Assist vacuum extension with Valetudo Robots in mind


# Valetudo voice control (rooms + passes) with View Assist map

Home Assistant blueprint (automation) for **Valetudo** robot vacuums from Dreame (maybe also from other brand, but i dont tested it) that works like the [Xiaomi Vacuum Map Card](https://github.com/sca075/lovelace-xiaomi-vacuum-map-card): you say which rooms and how many passes, Valetudo does it in **one run**.

| You say | What happens |
|---|---|
| "clean the kitchen twice" | one MQTT command, 2 passes |
| "clean kitchen and bedroom three times" | both rooms, 3 passes, in the spoken order |
| "show the vacuum" / "where is the robot" | opens your vacuum view on the View Assist display |
| "reinige die Küche zweimal" / "zeig mir den Staubsauger" | same in German (English and German are always both active) |
| your own sentences | optional, with your own actions (advanced) |

## How it works

The Xiaomi vacuum map card is frontend code, an automation cannot "press" it. What the card does when you
tap *Start* is publishing one MQTT message to Valetudo. The blueprint publishes exactly that:

```
<topic>/MapSegmentationCapability/clean/set
{"segment_ids": ["2","1"], "iterations": 3, "customOrder": true}
```

Room names come from Valetudo itself (retained topic `<topic>/MapData/segments`), so you
don't have to maintain segment IDs anywhere.

## Requirements

- Valetudo with MQTT enabled and Home Assistant's MQTT integration
- Optional: [View Assist](https://github.com/dinki/View-Assist) and
-  a vacuum map card + a map exposed as cam entity (e.g. Xiaomi Vacuum Map Card + Valetudo Vacuum Camera)
- Home Assistant 2024.10 or newer

## Install

1. import it with the button below **or** copy the file to `/config/blueprints/automation/<your-name>/` and reload.

   ```
   https://github.com/PsychoOW/Vacuum-Voice-View-Assist/blob/main/Blueprint%20-%20view%20assist%20-%20vacuum
   ```
2. Settings → Automations & scenes → Blueprints → *Valetudo voice control…* → **Create automation**.
3. Enter the Valetudo MQTT topic (Valetudo → Connectivity → MQTT, e.g.
   `valetudo/ShockingMustyReindeer`; it is the `topic` of `internal_variables` in your card).
4. Optional: pick the View Assist device and the view path.

## How spoken names are matched to rooms

In this order, case-insensitive:

1. **Extra room names** from the blueprint (`kitchen=2`), the explicit override
2. the **Valetudo segment name** ("Küche")
3. the **Home Assistant area** that the spoken word resolves to (name or alias, e.g. alias
   "kitchen" on the area "Küche") compared with the Valetudo segment name

Home Assistant keeps the segment→area mapping inside the entity options, which only the UI can
read, not automations. That is why names have to match or be listed under *Extra room names*.
Spoken names that match nothing get a spoken answer instead of a cleaning run.

## The View Assist view

A blueprint can't create dashboard views, so the map view is a few lines you paste once (just copy the card yaml from your xiaomi vacuum card if you use that)
(View Assist dashboard → edit → ⋮ → *Raw configuration editor*, under `views:`):

```yaml
- title: Vacuum
  path: vacuum
  type: panel
  cards:
    - type: custom:xiaomi-vacuum-map-card
      # ... your existing card configuration ...
```

`type: panel` makes the card fill the whole screen. Selecting rooms and passes by hand stays on
the card. The default view path of the blueprint is `/view-assist/vacuum`.

## Good to know

- **Conflicts:** Home Assistant runs every matching sentence trigger. The blueprint guards
  against double runs internally. Remove old custom sentences / intent scripts for
  "clean {room}" before you use it.
- **"clean the kitchen" (one pass)** keeps working through Assist's built-in intent. Switch on
  *Also handle plain "clean <room>"* if you want this blueprint to do it too (same message as the
  card, with map and spoken answer in your language).
- **customOrder:** on by default like the Xiaomi vacuum card. Turn it off if your firmware doesn't support it.
- **Valetudo only.** Other vacuums have no repeat option in Home Assistant's native
  `vacuum.clean_area` as far as i know; for them use Assist's built-in "clean <room>".
- One instance is enough. Several instances react to the same sentence.

## Own sentences (advanced)

Advanced → *Enable own sentences*, then add sentences and actions. Wildcards `{name}` end up in
`trigger.slots.name`, the full text is `trigger.sentence`. Answer with
`set_conversation_response`. Home Assistant requires at least one sentence, so a harmless
example is pre-filled.

```yaml
# sentences
- "start vacuum party {mode}"
# actions
- set_conversation_response: "Party mode {{ trigger.slots.mode }}"
```

## Tested

Tested against a real Home Assistant core (2026.2.3, headless) with a real MQTT broker: sentence
matching, retained segment list, name/area/alias/extra-name matching, several rooms with spoken
order, 1–3 passes, unknown rooms, missing room list, both languages and own sentences. The
published message is identical to the card's. **Not tested on a physical robot.**

And tested by me on a real robot with real home assistan on a real echo show 5 gen 1 running lineage os with view assist companion app installed and configured.

## Transparency disclosure
Love it or hate it, this was created with the help of AI. Which enables people like who are not specialized in computer/programming professions to create things like that.

---

## Kurzfassung (Deutsch)

Blueprint für View Assist in kombination mit Valetudo-Saugroboter: "reinige die Küche zweimal" (auch mehrere Räume, bis 3 Durchgänge) und "zeig mir den Staubsauger" (öffnet deine Karte auf dem View-Assist-Display). Der Blueprint sendet dieselbe MQTT-Nachricht wie die Xiaomi Vacuum Map Card, also ein einziger Lauf mit mehreren Durchgängen. Raumnamen liest er aus Valetudo (`MapData/segments`). Passt ein gesprochener Name nicht, trägst du ihn unter *Extra room names* als `name=segment-id` ein oder legst in Home Assistant einen Alias am Bereich an. Deutsche und englische Sätze sind immer beide aktiv, das Dropdown bestimmt nur die gesprochene Antwort. Alte eigene Sätze/Intent-Skripte für "clean {room}" vorher entfernen.
