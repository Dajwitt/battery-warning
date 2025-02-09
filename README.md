# 🔋 Home Assistant Batterie-Überwachung

Dieses Projekt ermöglicht eine automatische Überwachung von Batterien in Home Assistant. Es erkennt:

✅ Batterien, die unter 20% fallen und sendet eine Benachrichtigung.  
✅ Falls eine Batterie nach 2 Tagen nicht gewechselt wurde, erfolgt eine erneute Erinnerung.  
✅ Geräte, die keine Werte mehr senden (`unknown` oder `unavailable`).  
✅ Warnung, wenn ein Gerät nach dem Batteriewechsel keine neuen Werte sendet.  

## 🛠️ Voraussetzungen
- **Home Assistant** installiert
- **HACS (Home Assistant Community Store) muss installiert sein**
- **Integration `battery notes` über HACS installieren** ([Video-Anleitung](https://www.youtube.com/watch?v=D403Vy2VaFA&t=1278s))
- YAML-Modus für Automationen nutzen

## 📌 1️⃣ Sensoren & `battery notes` einrichten

1. **Gehe zu `Einstellungen → Geräte & Dienste → Integrationen`.**
2. **Suche `battery notes`** und installiere es über HACS.
3. **Sobald die Integration aktiv ist, werden `battery_plus`-Sensoren automatisch erstellt.**

## 📌 2️⃣ Sensor zur Erkennung niedriger Batterien einrichten

Füge den folgenden Code in deine `configuration.yaml` ein, um einen Template-Sensor zu erstellen, der die Anzahl der Batterien unter 20% zählt. **Dieser Sensor kann später im Dashboard angezeigt werden – dazu später mehr.**

```yaml
template:
  - sensor:
      - name: "battery_low_count"
        state: >-
          {% set batteries = states.sensor 
          | selectattr('entity_id', 'search', '_battery_plus$') 
          | selectattr('state', 'match', '^\\d+$') 
          | map(attribute='state') 
          | map('int') 
          | select('lt', 20) 
          | list %}
          {{ batteries | length }}
```

## 📌 3️⃣ Automationen zur Batterie-Überwachung hinzufügen

Füge folgende YAML-Automationen zu deiner `automations.yaml` hinzu:

- **Batterie unter 20% → Benachrichtigung senden & erneute Erinnerung nach 2 Tagen**  
  ```yaml
alias: Batterie unter 20% Warnung mit Erinnerung
description: ""
triggers:
  - value_template: >-
      {% set low_batteries = states.sensor  | selectattr('entity_id', 'search',
      '_battery_plus$')  | selectattr('state', 'match', '^\d+$')  |
      selectattr('state', 'lt', '20')  | list %} {{ low_batteries | length > 0
      }}
    trigger: template
  - value_template: >-
      {% set recovered_batteries = states.sensor  | selectattr('entity_id',
      'search', '_battery_plus$')  | selectattr('state', 'match', '^\d+$')  |
      selectattr('state', 'gt', '50')  | list %} {{ recovered_batteries | length
      > 0 }}
    trigger: template
conditions: []
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {% set batteries = states.sensor  | selectattr('entity_id',
              'search', '_battery_plus$')  | selectattr('state', 'match',
              '^\d+$')  | selectattr('state', 'lt', '20')  | list %} {{
              batteries | length > 0 }}
        sequence:
          - variables:
              low_batteries: >-
                {% set batteries = states.sensor  | selectattr('entity_id',
                'search', '_battery_plus$')  | selectattr('state', 'match',
                '^\d+$')  | selectattr('state', 'lt', '20')  |
                map(attribute='name')  | list %} {{ batteries }}
          - data:
              title: Batteriewarnung
              message: |
                {% if low_batteries | length == 1 %}
                  Die Batterie von {{ low_batteries[0] }} ist unter 20% gefallen!
                {% elif low_batteries | length > 1 %}
                  Die Batterien von {{ low_batteries | join(', ') }} sind unter 20% gefallen!
                {% else %}
                  Fehler: Keine Batterien unter 20% erkannt.
                {% endif %}
              notification_id: battery_low
            action: persistent_notification.create
          - data:
              title: Batterie-Warnung
              message: |
                {% if low_batteries | length == 1 %}
                  Die Batterie von {{ low_batteries[0] }} ist unter 20% gefallen!
                {% elif low_batteries | length > 1 %}
                  Die Batterien von {{ low_batteries | join(', ') }} sind unter 20% gefallen!
                {% else %}
                  Fehler: Keine Batterien unter 20% erkannt.
                {% endif %}
              data:
                tag: battery_low
                sticky: true
            action: notify.mobile_app_galaxy_s23
          - delay: "48:00:00"
          - condition: template
            value_template: >-
              {% set still_low_batteries = states.sensor  |
              selectattr('entity_id', 'search', '_battery_plus$')  |
              selectattr('state', 'match', '^\d+$')  | selectattr('state', 'lt',
              '20')  | list %} {{ still_low_batteries | length > 0 }}
          - data:
              title: "Erinnerung: Batterie weiterhin unter 20%"
              message: |
                {% if still_low_batteries | length == 1 %}
                  Die Batterie von {{ still_low_batteries[0] }} ist weiterhin unter 20%! Bitte wechseln.
                {% elif still_low_batteries | length > 1 %}
                  Die Batterien von {{ still_low_batteries | join(', ') }} sind weiterhin unter 20%! Bitte wechseln.
                {% else %}
                  Fehler: Keine Batterien unter 20% erkannt.
                {% endif %}
              data:
                tag: battery_low_reminder
                sticky: true
            action: notify.mobile_app_galaxy_s23
      - conditions:
          - condition: template
            value_template: >-
              {% set recovered_batteries = states.sensor  |
              selectattr('entity_id', 'search', '_battery_plus$')  |
              selectattr('state', 'match', '^\d+$')  | selectattr('state', 'gt',
              '50')  | list %} {{ recovered_batteries | length > 0 }}
        sequence:
          - data:
              notification_id: battery_low
            action: persistent_notification.dismiss
mode: restart

```

- **Warnung, wenn ein Sensor `unavailable` oder `unknown` wird**  
  (Datei: [`automations.yaml`](automations.yaml))

- **Erkennung, wenn ein Gerät nach einem Batteriewechsel keine Werte sendet**  
  (Datei: [`automations.yaml`](automations.yaml))

## 🎯 Fazit
Mit diesem Projekt hast du eine **automatische, wartungsfreie Batterie-Überwachung** in Home Assistant. Falls du Verbesserungen hast, erstelle gerne einen Pull-Request! 🚀
