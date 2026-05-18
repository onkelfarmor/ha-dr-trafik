# DR Trafik – Home Assistant Integration

Henter trafikdata fra **Vejdirektoratets officielle NAP API** – samme datakilde som [dr.dk/trafik](https://www.dr.dk/trafik).

---

## Funktioner

- **3 sensor-entiteter** pr. opsætning:
  - `sensor.dr_trafik_aktive_meldinger_i_alt` – samlet tæller
  - `sensor.dr_trafik_trafikhaendelser` / `vejarbejde` / `vinter...` – pr. type
  - `sensor.dr_trafik_seneste_melding` – seneste overskrift som tilstand
- **Filtrering efter region** (Hele landet, Kbh/Sjælland, Jylland, Fyn/Sydjylland)
- **Automatisk event** (`dr_trafik_new_incident`) ved nye hændelser → bruges til push-notifikationer
- **Konfigurerbart opdateringsinterval** (1–60 min, standard 5 min)
- **Options-flow** så du kan ændre indstillinger uden at slette og geninstallere

---

## Installation

### HACS (anbefalet)

1. Åbn HACS → **Integrationer** → Klik `⋮` øverst til højre → **Brugerdefinerede arkiver**
2. Tilføj URL: `https://github.com/DIN-BRUGER/dr_trafik` (skift til dit repo)
3. Installer "DR Trafik" via HACS
4. Genstart Home Assistant

### Manuel installation

1. Kopiér mappen `custom_components/dr_trafik/` til din HA `config/custom_components/`
2. Genstart Home Assistant

---

## Opsætning

### Trin 1 – Hent API-nøgle (gratis)

1. Gå til **[https://nap.vd.dk/themes/1](https://nap.vd.dk/themes/1)**
2. Opret konto (kræver kun e-mail)
3. Generér en API-nøgle under **"Min profil"**

### Trin 2 – Tilføj integrationen i HA

1. Gå til **Indstillinger → Enheder & tjenester → Tilføj integration**
2. Søg efter **"DR Trafik"**
3. Udfyld formularen:

| Felt | Beskrivelse |
|------|-------------|
| NAP API-nøgle | Din nøgle fra nap.vd.dk |
| Region | Vælg ønsket geografisk filtrering |
| Opdateringsinterval | Hvor tit API'et spørges (min. 1 min) |
| Datatyper | Trafikhændelser, Vejarbejde, Vintervejforhold, Glatførebekæmpelse |
| Affyr event | Til/fra for push-notifikationer |

---

## Push-notifikationer

Integrationen affyrer et HA-event **`dr_trafik_new_incident`** hver gang en ny hændelse registreres.

Event-data indeholder:
```yaml
heading: "Ulykke på E45 ved Aarhus"
description: "Vognbane spærret i nordgående retning. Kø 3 km."
entity_type: "ACCIDENT_BLOCKING"
tag: "TR-12345"
timestamp: "2026-01-15T08:23:00Z"
region: "midt_nord_ostjylland"
is_critical: true
```

Se `automations_eksempler.yaml` for færdige automationer til:
- Push til mobil-app (iOS + Android)
- Kritiske varsler med høj prioritet
- Daglig morgenbriefing
- Persistent notification i HA UI
- TTS-annoncering på Google Home / Alexa
- Telegram-beskeder

### Hurtigstart – Push til mobil

```yaml
automation:
  - alias: "DR Trafik push"
    trigger:
      - platform: event
        event_type: dr_trafik_new_incident
    action:
      - service: notify.mobile_app_din_telefon
        data:
          title: "🚦 {{ trigger.event.data.heading }}"
          message: "{{ trigger.event.data.description }}"
```

---

## Sensor-attributter

### `sensor.dr_trafik_trafikhaendelser`

```yaml
state: 4  # antal aktive hændelser
attributes:
  meldinger:
    - overskrift: "Ulykke på motorvejen"
      beskrivelse: "Spærret i venstre vognbane"
      kategori: "ACCIDENT"
      tidsstempel: "2026-01-15T08:23:00Z"
      tag: "TR-12345"
  total: 4
```

### `sensor.dr_trafik_seneste_melding`

```yaml
state: "Ulykke på E45 ved Aarhus"
attributes:
  aktiv: true
  overskrift: "Ulykke på E45 ved Aarhus"
  beskrivelse: "Vognbane spærret. Kø 3 km."
  kategori: "ACCIDENT_BLOCKING"
  tidsstempel: "2026-01-15T08:23:00Z"
```

---

## Lovelace-kort

```yaml
# Enkelt overblikskort
type: entities
title: Trafik
entities:
  - entity: sensor.dr_trafik_aktive_meldinger_i_alt
  - entity: sensor.dr_trafik_trafikhaendelser
  - entity: sensor.dr_trafik_vejarbejde
  - entity: sensor.dr_trafik_seneste_melding

# Markdown-kort med liste over meldinger
type: markdown
title: Aktive trafikmeldinger
content: >
  {% set meldinger = state_attr('sensor.dr_trafik_trafikhaendelser', 'meldinger') %}
  {% if meldinger %}
    {% for m in meldinger %}
  **{{ m.overskrift }}**
  {{ m.beskrivelse }}
  ---
    {% endfor %}
  {% else %}
  Ingen aktive meldinger 🟢
  {% endif %}
```

---

## Datakilder og licens

Trafikdata leveres af [Vejdirektoratet](https://www.vejdirektoratet.dk/) via
[NAP (National Access Point)](https://nap.vd.dk/).
DR.dk/trafik bruger samme datakilde.

API-dokumentation: [github.com/Vejdirektoratet/sdk-web](https://github.com/Vejdirektoratet/sdk-web)

Integrationen er ikke officielt tilknyttet DR eller Vejdirektoratet.
