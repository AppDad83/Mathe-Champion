# Mathe-Champion App Architecture (Nordic Mythology Theme)

## 1. Tech Stack

| Kategorie | Technologie |
|-----------|-------------|
| Frontend | Vanilla JavaScript (ES6+), kein Framework |
| Styling | Inline CSS in `<style>`-Block, CSS Custom Properties |
| Native Wrapper | Capacitor 5.7.0 |
| Persistenz | `@capacitor/preferences` mit localStorage-Fallback |
| Fonts | Google Fonts (MedievalSharp für Überschriften, Nunito für Body) |
| 3D-Viewer | Three.js r128 + GLTFLoader (dynamisch geladen) |
| Build | Kein Build-Prozess ("static files only") |
| Android | Gradle, compileSdk/targetSdk via `variables.gradle` |

**Dependencies (package.json):**
```json
"@capacitor/android": "^5.7.0",
"@capacitor/core": "^5.7.0",
"@capacitor/preferences": "^5.0.7"
```

---

## 2. Dateistruktur

```
Mathe_3_H2_2/
├── www/
│   ├── index.html              # Single-File-App (HTML + CSS + JS)
│   └── assets/
│       ├── characters/         # PNG Charakterbilder (512x512)
│       │   └── .gitkeep
│       ├── models/             # GLB 3D-Modelle
│       │   └── .gitkeep
│       └── README.md           # Asset-Dokumentation
├── android/
│   ├── app/
│   │   ├── build.gradle
│   │   └── src/main/
│   │       ├── AndroidManifest.xml
│   │       └── res/
│   ├── gradle.properties
│   ├── settings.gradle
│   ├── variables.gradle
│   └── gradlew.bat
├── capacitor.config.json       # appId: de.sophia.mathetrainer
├── package.json
└── node_modules/
```

---

## 3. Design-System (Nordic Theme)

### CSS Custom Properties (`:root`)

| Variable | Wert | Verwendung |
|----------|------|------------|
| `--primary` | `#1B3A6B` | Dunkelblau (Hauptfarbe) |
| `--primary-light` | `#2E5FA3` | Hover-States |
| `--primary-dark` | `#122649` | Gedrückt-States |
| `--accent` | `#B8860B` | Gold (Sterne, Buttons, Akzente) |
| `--accent-light` | `#D4A017` | Leuchtendes Gold |
| `--background` | `#0D1B2A` | Body-Hintergrund (Dunkel) |
| `--surface` | `#152238` | Sekundärer Hintergrund |
| `--success` | `#2D6A4F` | Richtige Antworten |
| `--error` | `#9B2335` | Falsche Antworten |
| `--card-bg` | `#1A2F4A` | Karten-Hintergrund |
| `--text` | `#F0E6D3` | Haupttext (Warm Off-White) |
| `--text-light` | `#A89F8C` | Sekundärtext |

### Typography

- **Überschriften:** `'MedievalSharp', cursive`
- **Body-Text:** `'Nunito', sans-serif`
- **Base Size:** 18px
- **Weights:** 400, 600, 700, 800

### Dekorative Klassen

| Klasse | Beschreibung |
|--------|--------------|
| `.rune-divider` | Goldene Linie mit Runen ᚠ ᚢ ᚦ ᚬ ᚱ ᚴ |
| `.norse-card` | Karte mit goldenem Border und Runen-Wasserzeichen |
| `.prophecy-box` | Countdown-Box im Prophezeiung-Stil |

---

## 4. Reward/Gamification-System

### Sterne-Logik (unverändert)

| Prozent korrekt | Sterne |
|-----------------|--------|
| ≥ 90% | ⭐⭐⭐ |
| ≥ 80% | ⭐⭐ |
| ≥ 60% | ⭐ |
| < 60% | 0 |

### Norse Unlocks System (NEU)

**Datenstruktur** (`NORSE_UNLOCKS`):
```javascript
{
    id: string,           // Eindeutige ID
    type: 'character' | 'item',
    starsRequired: number, // Benötigte Gesamtsterne
    name: string,
    subtitle: string,
    emoji: string,
    runeTitle: string,    // Nordische Runen
    description: string,
    image?: string,       // Pfad zu PNG (Charaktere)
    model?: string,       // Pfad zu GLB (Items)
    parentCharacter?: string, // Verknüpfung Item → Charakter
    secret?: boolean      // Versteckt bis freigeschaltet
}
```

**Freischalt-Reihenfolge:**
| Sterne | Unlock |
|--------|--------|
| 5 | Thor ⚡ |
| 8 | Mjölnir 🔨 |
| 12 | Loki 🐺 |
| 15 | Andvaranaut 💍 |
| 19 | Odin 🦅 |
| 22 | Gungnir 🔱 |
| 27 | Heimdall 🌈 |
| 29 | Gjallarhorn 📯 |
| 33 | Freya ⚔️ (Secret) |
| 33 | Brísingamen 📿 (Secret) |

### State-Management

```javascript
const state = {
    topicStars: {},       // { topicId: stars }
    unlockedIds: [],      // ['thor', 'mjolnir', ...]
    // ...
};
```

### Storage Keys

| Key | Inhalt |
|-----|--------|
| `topicStars` | JSON `{"vielfache":3,"teiler":2,...}` |
| `norseUnlocks` | JSON `["thor","mjolnir",...]` |

### Relevante Funktionen

| Funktion | Beschreibung |
|----------|--------------|
| `getTotalStars()` | Berechnet Summe aller Sterne |
| `checkAndUnlock()` | Prüft neue Freischaltungen nach Session |
| `showUnlockAnimation(unlock)` | Zeigt goldene Funken-Animation |
| `getNextUnlock()` | Gibt nächsten freischaltbaren Unlock zurück |

---

## 5. Komponenten-Übersicht

| Komponente | Funktion |
|------------|----------|
| **Storage** | Capacitor Preferences + localStorage Fallback |
| **Topic Grid** | 2-Spalten Grid mit Runen-Icons pro Thema |
| **Question Generator** | Factory für 11 Aufgabentypen |
| **Input Renderer** | numpad, multichoice, yesno, multiselect |
| **Scratch Pad** | Canvas-Zeichenfläche, goldene Stiftfarbe |
| **Hint System** | Zwei-stufige Tipps |
| **Answer Checker** | Validierung mit Toleranz-Option |
| **Animations** | Star-Burst, goldenes Confetti |
| **Hall Screen** | Götter-Galerie mit Fortschrittsanzeige |
| **3D Viewer** | Three.js GLB-Viewer mit Touch-Rotation |
| **Unlock Animation** | Goldene Funken + Overlay |

---

## 6. Entry Points & Screens

### Haupt-Einstiegspunkt

**Datei:** `www/index.html`

**Boot-Sequenz:**
```
1. DOMContentLoaded
2. initApp()
3. loadProgress() → lädt topicStars + norseUnlocks
4. renderTopicGrid()
5. updateCountdown()
6. updateHallBadge()
```

### Screen-System

| Screen ID | Aktivierung |
|-----------|-------------|
| `home-screen` | Initial, `goHome()` |
| `practice-screen` | `startTopic(id)`, `startExamMode()` |
| `end-screen` | `endSession()` |
| `hall-screen` | `showHallScreen()` |

### Globale Funktionen (window.*)

```javascript
// Navigation
window.goHome
window.showHallScreen
window.startTopic
window.startExamMode

// Input
window.pressKey
window.submitNumpad
window.selectChoice
window.submitYesNo
window.toggleMultiSelect
window.submitMultiSelect
window.nextQuestion
window.showHint
window.clearScratchPad

// Unlock System
window.closeUnlockOverlay
window.closeUnlockAndGoHall
window.show3DViewer
window.close3DViewer

// Utilities
window.Storage
window.initApp
```

---

## 7. 3D-Viewer Details

### Ladeprozess

1. `show3DViewer(itemId)` aufrufen
2. Dynamisches Laden von Three.js + GLTFLoader
3. Szene mit dunklem Hintergrund + goldenem Ambient Light
4. GLB-Modell laden und zentrieren
5. Manuelles Rotate via Touch/Mouse

### Fallback

Wenn GLB nicht ladbar:
- Zeigt großes Emoji
- Name + Beschreibung
- Keine Fehlermeldung

### Touch-Steuerung (kein OrbitControls)

```javascript
canvas.addEventListener('touchmove', (e) => {
    viewerMesh.rotation.y += deltaX * 0.01;
});
```

---

## Schnellreferenz für Änderungen

| Task | Datei/Zeile |
|------|-------------|
| Neues Thema hinzufügen | `topics` Array + `TOPIC_RUNES` + `create*Question()` |
| Neuen Charakter/Item | `NORSE_UNLOCKS` Array |
| Farben ändern | `:root` CSS Variables |
| Sterne-Schwellwerte | `endSession()` |
| 3D-Modell hinzufügen | `www/assets/models/` + `NORSE_UNLOCKS.model` |
| Charakterbild | `www/assets/characters/` |
