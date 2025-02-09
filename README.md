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
4. **Drücke den Button `button.*_battery_replaced`, um den Wechsel zu speichern.**
5. **Der letzte Batteriewechsel wird in `sensor.*_battery_last_replaced` gespeichert.**

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

## 📌 3️⃣ Automationen zur Batterie-Überwachung erstellen

💡 **Hinweis:** Ersetze `notify.mobile_app_galaxy_s23` in den Automationen mit deinem eigenen Home Assistant Benachrichtigungsdienst. Dies findest du unter `Einstellungen → Geräte & Dienste → Dienste`.

### Automationen in Home Assistant erstellen

1. **Wechsle in den YAML-Modus für Automationen:**
   - Gehe zu `Einstellungen → Automatisierungen`.
   - Klicke auf `Automationen` und dann auf das `+` Symbol, um eine neue Automation zu erstellen.
   - Wähle `Bearbeiten in YAML`, um den YAML-Modus zu aktivieren.
   
2. **Füge die folgende YAML-Konfiguration in die neue Automation ein:**

- **Batterie unter 20% → Benachrichtigung senden & erneute Erinnerung nach 2 Tagen**
  (Datei: [`automation_battery_low.yaml`](automation_battery_low.yaml))

- **Warnung, wenn ein Sensor `unavailable` oder `unknown` wird**  
  (Datei: [`automation_battery_unavailable.yaml`](automation_battery_unavailable.yaml))

- **Erkennung, wenn ein Gerät nach einem Batteriewechsel keine Werte sendet**  
  (Datei: [`automation_battery_replacement.yaml`](automation_battery_replacement.yaml))
