# SuperGlue Image Matching - Dokumentation

## Übersicht
Dieses Notebook verwendet **SuperGlue** (MagicLeap) für präzises Image Matching zwischen zwei Bildern. SuperGlue kombiniert **SuperPoint** (Keypoint Detection) mit einem Graph Neural Network für robustes Feature Matching.

---

## 📋 Zelle 1: Bild-Pfade

```python
osm_png_path = "..."           # Referenzbild (z.B. OSM Karte)
bplan_png_path = "..."         # Zielbild 1 (B-Plan)
liegenschaft_png_path = "..."  # Zielbild 2 (Liegenschaftskataster)
```

**Was einstellen:**
- Ändere die Pfade zu deinen eigenen Bildern

---

## ⚙️ Zelle 2: SuperGlue Matching

### **PARAMETER 1: Bildauflösung**
```python
IMAGE_SIZE = 640  # Pixel (Breite & Höhe)
```
- **256** = sehr schnell, weniger genau
- **640** = guter Kompromiss (empfohlen)
- **1024** = langsam aber sehr genau

### **PARAMETER 2: SuperPoint Konfiguration**
```python
config = {
    'superpoint': {
        'nms_radius': 4,                    # Non-Maximum Suppression Radius
        'keypoint_threshold': 0.005,        # Schwellwert für Keypoint-Detektion
        'max_keypoints': 1024               # Maximale Anzahl Keypoints pro Bild
    }
}
```

**Was einstellen:**
- `keypoint_threshold`: **0.001** bis **0.01**
  - Niedriger = mehr Keypoints (aber auch mehr Rauschen)
  - Höher = weniger aber qualitativ bessere Keypoints
  - Empfohlen: **0.005**

- `max_keypoints`: **512** bis **2048**
  - Mehr = genauer aber langsamer
  - Empfohlen: **1024**

### **PARAMETER 3: SuperGlue Matching**
```python
config = {
    'superglue': {
        'weights': 'outdoor',               # Modell-Typ
        'sinkhorn_iterations': 20,          # Matching-Iterationen
        'match_threshold': 0.2,             # Confidence-Schwellwert
    }
}
```

**Was einstellen:**
- `weights`: 
  - **'outdoor'** für Karten, Luftbilder, Outdoor-Szenen (empfohlen)
  - **'indoor'** nur wenn beide Bilder Indoor-Aufnahmen sind

- `sinkhorn_iterations`: **20** bis **100**
  - Mehr = genauer aber langsamer
  - Empfohlen: **20**

- `match_threshold`: **0.0** bis **0.9**
  - **0.2** = Standard (akzeptiert viele Matches)
  - **0.5** = Strenger (nur gute Matches)
  - **0.8** = Sehr streng (sehr wenige Matches)

### **PARAMETER 4: Maximale Anzahl Matches**
```python
MAX_MATCHES = 100  # Limitierung auf TOP-N Matches
```

**Was einstellen:**
- **50** = schnelle Visualisierung, weniger Matches
- **100** = guter Kompromiss (empfohlen)
- **200-500** = viele Matches
- **999999** = alle verfügbaren Matches verwenden

**Wichtig:** Es werden automatisch die Matches mit der **höchsten Confidence** ausgewählt!

---

## 📊 Zelle 3: Match-Details (Optional)

```python
DISPLAY_THRESHOLD = 0.5  # Nur Matches mit Confidence > 0.5 anzeigen
```

**Was einstellen:**
- **0.3** = zeigt mehr Matches
- **0.5** = guter Kompromiss (empfohlen)
- **0.7** = nur sehr sichere Matches

Diese Zelle zeigt die ersten 10 Matches mit ihren Koordinaten und Confidence-Scores an.

---

## 🎨 Zelle 4: Visualisierung

```python
VISUALIZATION_THRESHOLD = 0.2  # Welche Matches visualisiert werden
```

**Was einstellen:**
- **0.0** = zeigt ALLE Matches (kann unübersichtlich werden)
- **0.2** = empfohlener Wert
- **0.5** = nur gute Matches
- **0.8** = nur sehr sichere Matches

**Farben:**
- Alle Matches sind **grün** 🟢
- Keine Farb-Kodierung nach Confidence

**Ausgabe:**
- Beide Bilder nebeneinander
- Grüne Linien zwischen gematchten Keypoints
- Info-Box mit Anzahl und durchschnittlicher Confidence

---

## 🎯 Workflow

1. **Zelle 1 ausführen** → Bild-Pfade setzen
2. **Zelle 2 anpassen** → Parameter einstellen, dann ausführen
   - Dauert 5-30 Sekunden je nach `IMAGE_SIZE`
3. **Optional: Zelle 3** → Zeigt Match-Details
4. **Zelle 4 ausführen** → Visualisierung anzeigen

---

## 💡 Tipps für beste Ergebnisse

### Für **viele Matches**:
```python
keypoint_threshold = 0.001      # Mehr Keypoints
max_keypoints = 2048            # Höheres Limit
match_threshold = 0.2           # Akzeptiert mehr Matches
MAX_MATCHES = 999999            # Keine Limitierung
```

### Für **qualitativ beste Matches**:
```python
keypoint_threshold = 0.01       # Nur gute Keypoints
match_threshold = 0.5           # Strenge Matches
MAX_MATCHES = 100               # Top 100
```

### Für **schnelle Verarbeitung**:
```python
IMAGE_SIZE = 512                # Kleinere Auflösung
max_keypoints = 512             # Weniger Keypoints
MAX_MATCHES = 50                # Weniger Matches
```

---

## 📝 Alle einstellbaren Parameter auf einen Blick

| Parameter | Wo? | Bereich | Empfohlen | Effekt |
|-----------|-----|---------|-----------|--------|
| `IMAGE_SIZE` | Zelle 2 | 256-1024 | 640 | Bildauflösung |
| `keypoint_threshold` | Zelle 2 | 0.001-0.01 | 0.005 | Keypoint-Qualität |
| `max_keypoints` | Zelle 2 | 512-2048 | 1024 | Max. Keypoints/Bild |
| `weights` | Zelle 2 | indoor/outdoor | outdoor | Modell-Typ |
| `match_threshold` | Zelle 2 | 0.0-0.9 | 0.2 | Match-Strenge |
| `MAX_MATCHES` | Zelle 2 | 50-999999 | 100 | Anzahl Matches |
| `DISPLAY_THRESHOLD` | Zelle 3 | 0.0-1.0 | 0.5 | Text-Ausgabe Filter |
| `VISUALIZATION_THRESHOLD` | Zelle 4 | 0.0-1.0 | 0.2 | Plot-Filter |

---

## ⚠️ Häufige Probleme

**Problem:** Zu wenige Matches
- Lösung: `match_threshold` senken (z.B. auf 0.1)
- Lösung: `keypoint_threshold` senken (mehr Keypoints)

**Problem:** Zu viele falsche Matches
- Lösung: `match_threshold` erhöhen (z.B. auf 0.5)
- Lösung: `keypoint_threshold` erhöhen (bessere Keypoints)

**Problem:** Langsam
- Lösung: `IMAGE_SIZE` reduzieren
- Lösung: `max_keypoints` reduzieren

---

## 📁 Projektstruktur

```
LoFTR_map_georeferencer/
├── superglue_test.ipynb          # Hauptnotebook
├── SUPERGLUE_README.md           # Diese Datei
├── LoFTR_map_georeferencer/
│   ├── models/
│   │   ├── matching.py           # SuperGlue + SuperPoint Wrapper
│   │   ├── superglue.py          # SuperGlue Implementierung
│   │   ├── superpoint.py         # SuperPoint Keypoint Detector
│   │   ├── utils.py              # Hilfsfunktionen
│   │   └── weights/              # Pre-trained Models
│   │       ├── superglue_outdoor.pth
│   │       ├── superglue_indoor.pth
│   │       └── superpoint_v1.pth
└── output/                        # Output Ordner für Ergebnisse
```

---

Viel Erfolg! 🚀
