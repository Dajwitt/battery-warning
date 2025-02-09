# 🔋 Home Assistant Batterie-Überwachung

Home Assistant bietet viele Möglichkeiten zur Überwachung von Geräten – doch was ist mit den **Batterieständen**?

Mit diesem Projekt kannst du ganz einfach den Batteriestatus aller Geräte überwachen. Es erkennt:

✅ Batterien, die unter 20% fallen und sendet eine Benachrichtigung.  
✅ Falls eine Batterie nach 2 Tagen nicht gewechselt wurde, erfolgt eine erneute Erinnerung.  
✅ Geräte, die keine Werte mehr senden (`unknown` oder `unavailable`).  
✅ Warnung, wenn ein Gerät nach dem Batteriewechsel keine neuen Werte sendet.  

## 🛠️ Voraussetzungen
- **Home Assistant** installiert
- **HACS (Home Assistant Community Store) muss installiert sein**
- **Integration `battery notes` über HACS installieren** ([Video-Anleitung von Smart-Live](https://www.youtube.com/watch?v=D403Vy2VaFA&t=1278s))
- YAML-Modus für Automationen nutzen

## 📌 1️⃣ Sensoren & `battery notes` einrichten

1. **Gehe zu `Einstellungen → Geräte & Dienste → Integrationen`.**
2. **Suche `battery notes`** und installiere es über HACS.
3. **Sobald die Integration aktiv ist, werden `battery_plus`-Sensoren automatisch erstellt.**
4. **Drücke den Button `button.*_battery_replaced`, um den Wechsel zu speichern.**
5. **Der letzte Batteriewechsel wird in `sensor.*_battery_last_replaced` gespeichert.**

## 📌 2️⃣ Sensor zur Erkennung niedriger Batterien einrichten

Füge den folgenden Code in deine `configuration.yaml` ein, um einen Template-Sensor zu erstellen, der die Anzahl der Batterien unter 20% zählt. 
**Dieser Sensor zählt automatisch alle `battery_plus`-Sensoren, die unter 20% fallen, und kann für Automationen sowie im Dashboard genutzt werden.**

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

## 📌 3️⃣ Automationen zur Batterie-Überwachung erstellen

Diese drei Automationen sorgen für eine vollständige Batterie-Überwachung:

In diesem Projekt gibt es **drei Automationen**, die zusammen für eine vollständige Batterie-Überwachung sorgen:
1. **Warnung bei Batterien unter 20% mit erneuter Erinnerung nach 2 Tagen**
2. **Erkennung von Sensoren, die keine Werte mehr liefern (`unknown` oder `unavailable`)**
3. **Warnung, wenn ein Gerät nach einem Batteriewechsel keine Werte sendet**

💡 **Hinweis:** Ersetze `notify.mobile_app_galaxy_s23` in den Automationen mit deinem eigenen Home Assistant Benachrichtigungsdienst. Dies findest du unter `Einstellungen → Geräte & Dienste → Dienste`.

### 📌 So fügst du die Automationen in Home Assistant hinzu

1. **Wechsle in den YAML-Modus für Automationen:**
   - Gehe zu `Einstellungen → Automatisierungen`.
   - Klicke auf `Automationen` und dann auf das `+` Symbol, um eine neue Automation zu erstellen.
   - Wähle `Bearbeiten in YAML`, um den YAML-Modus zu aktivieren.
   
2. **Füge den passenden YAML-Code in die neue Automation ein:**
   - Öffne das YAML-Feld und kopiere den Inhalt der jeweiligen Datei (`automation_battery_low.yaml`, etc.).
   - Klicke auf `Speichern`.
   - Wiederhole dies für alle Automationen.

- Automation 1: **Batterie unter 20% → Benachrichtigung senden & erneute Erinnerung nach 2 Tagen**  
  (Datei: [`automation_battery_low.yaml`](automation_battery_low.yaml)) 

- Automation 2: **Warnung, wenn ein Sensor `unavailable` oder `unknown` wird**  
  (Datei: [`automation_battery_unavailable.yaml`](automation_battery_unavailable.yaml))

- Automation 3: **Erkennung, wenn ein Gerät nach einem Batteriewechsel keine Werte sendet**  
  (Datei: [`automation_battery_replacement.yaml`](automation_battery_replacement.yaml))

## 📌 4️⃣ Komplette Batterie-Übersicht im Dashboard erstellen

Damit du die Batterie-Überwachung optimal visualisieren kannst, gibt es drei verschiedene Anzeigeoptionen:

✅ Die **Gesamtanzahl der Batterien unter 20%**  
✅ Eine **Liste aller Batterien unter 20%**  
✅ Eine **grafische Darstellung des Batteriestatus wie auf dem Smartphone als Badge**
✅ Die **Gesamtanzahl der Batterien unter 20%**
✅ Eine **Liste aller Batterien unter 20%**
✅ Eine **grafische Darstellung des Batteriestatus wie auf dem Smartphone als Batch**


[![Beschreibung des Bildes](https://github.com/dajwitt/battery-warning/raw/main/images/batterie_dashboard.png)](https://github.com/Dajwitt/battery-warning/blob/main/Screenshot%202025-02-09%2009.03.12.png)](https://github.com/Dajwitt/battery-warning/blob/main/Screenshot%202025-02-09%2009.03.12.png?raw=true)


### 📌 Schritte zur Einrichtung

Damit du die Batterie-Überwachung optimal visualisieren kannst, zeige ich dir drei verschiedenen Anzeigeoptionen. Diese zeigen dir:
✅ Die **Gesamtanzahl der Batterien unter 20%**
✅ Eine **Liste aller Batterien unter 20%**
✅ Eine **grafische Darstellung des Batteriestatus wie auf dem Smartphone als Batch**

### **📊 1️⃣ Tile Card: Zeigt die Anzahl der Batterien unter 20%**

```yaml
type: vertical-stack
cards:
  - type: tile
    entity: sensor.battery_low_count
    name: Batterien unter 20%
    icon: mdi:battery-low
    grid_options:
      columns: 6  
```
### **📋 2️⃣ Auto-Entities Karte: Zeigt eine Liste aller Batterien unter 20%**

```yaml
type: custom:auto-entities
card:
  type: entities
  title: Batterien unter 20%
filter:
  include:
    - entity_id: /.*_battery_plus$/
      state: < 20
icon: mdi:battery-20
color: red
```

### **📊 3️⃣ Mushroom Template Card: Kompakte Anzeige mit Icon**

```yaml
type: custom:mushroom-template-card
entity: sensor.battery_low_count
icon: mdi:battery-low
layout: vertical
alignment: right
fill_container: false
badge_icon: |-
  {% set count = states(entity) | int(0) %}
  {{ 'mdi:numeric-' ~ count if count < 9 else 'mdi:numeric-9-plus' }}
icon_color: blue
badge_color: |-
  {% if states(entity) | int > 0 %}
    red
  {% else %}
    green
  {% endif %}
card_mod:
  style: |
    ha-card { background: transparent;
      --border-style: none;
      --border: 0px;
      --ha-card-header-font-size: 20px;
      --bar-card-border-radius: 51px;
      --ha-card-border-width: 0px;
    }
    mushroom-badge-icon {
      --badge-icon-size: 27px;
      --badge-size: 20px;
    }
```

Diese drei Methoden bieten verschiedene Möglichkeiten, die Batterie-Überwachung direkt in dein Home Assistant Dashboard zu integrieren.

## 🎯 Fazit: Nie wieder leere Batterien verpassen!

Mit diesem Projekt hast du eine **automatische, wartungsfreie Batterie-Überwachung** in Home Assistant. 
🔹 **Sofortige Benachrichtigung, wenn eine Batterie schwach wird.**
🔹 **Erneute Erinnerung nach 2 Tagen, falls du den Wechsel vergisst.**
🔹 **Erkennung von Geräten, die keine Werte mehr senden oder nach einem Wechsel nicht reagieren.**

📢 **Falls du Ideen oder Verbesserungen hast, erstelle gerne einen Pull-Request oder teile dein Feedback!** 🚀
