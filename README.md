# 🫧 Bubble Experiment 🧪

> Ein interaktives Mini-Projekt, das spielerisch das Konzept von Filterblasen erkundet. Wähle Wörter aus, die dich ansprechen, und finde heraus, in welcher "Bubble" du landest!

Dieses Projekt wurde mit [SvelteKit](https://kit.svelte.dev/) erstellt.

## 🌟 Features

*   **Interaktives Quiz:** Navigiere durch 10 Runden und triff Entscheidungen durch Wortauswahl.
*   **Dynamische Filterblasen:** Deine Auswahl bestimmt, ob du in der Technologie- 🧑‍💻, Klima- 🌍, Kultur- 🎨 oder Wirtschafts-Bubble 💰 landest.
*   **Visuelle Effekte:** Jede Bubble hat am Ende einen einzigartigen Partikeleffekt (z.B. Blätter 🍃 für Klima, Nullen und Einsen für Technologie, Münzen für Wirtschaft).
*   **Responsives Design:** Angepasst für eine gute Darstellung auf verschiedenen Bildschirmgrößen.

## 🤔 Worum geht es?

Das "Bubble Experiment" ist eine kleine Webanwendung, bei der Benutzer durch eine Reihe von Wortauswahlen geführt werden. Jede Auswahl beeinflusst, welcher von vier Kategorien (Technologie, Klima, Kultur, Wirtschaft) der Benutzer zugeneigt ist. Nach 10 Runden wird die dominante Kategorie als "Filterblase" des Benutzers identifiziert und mit einer passenden Beschreibung und einem thematischen Partikeleffekt visualisiert. Ziel ist es, auf unterhaltsame Weise zu zeigen, wie sich Präferenzen zu einer thematischen Eingrenzung verdichten können.

## 🛠️ Verwendete Technologien

*   **SvelteKit:** Framework für die Entwicklung der Webanwendung.
*   **TypeScript:** Für typsicheren JavaScript-Code.
*   **HTML5 Canvas:** Zur Darstellung der Partikeleffekte.
*   **CSS3:** Für das Styling der Anwendung.

## 🚀 Loslegen

### Voraussetzungen

*   [Node.js](https://nodejs.org/) (Version 18.x oder höher empfohlen)
*   npm, pnpm oder yarn

### Projekt erstellen (Falls du es neu aufsetzen möchtest)

```bash
# Erstelle ein neues SvelteKit-Projekt (folge den Anweisungen)
npm create svelte@latest bubble-experiment
cd bubble-experiment
```

### Abhängigkeiten installieren

Wenn du das Projekt bereits geklont hast, navigiere in das Projektverzeichnis und installiere die Abhängigkeiten:

```bash
npm install
# oder
pnpm install
# oder
yarn
```

### Entwicklungsserver starten

Starte den Entwicklungsserver. Die Anwendung ist dann normalerweise unter `http://localhost:5173` erreichbar.

```bash
npm run dev

# Oder starte den Server und öffne die App direkt in einem neuen Browser-Tab
npm run dev -- --open
```

### Produktions-Build erstellen

Um eine optimierte Version der Anwendung für das Deployment zu erstellen:

```bash
npm run build
```

Du kannst den Produktions-Build lokal mit `npm run preview` testen.

> Für das Deployment deiner Anwendung benötigst du möglicherweise einen [SvelteKit Adapter](https://kit.svelte.dev/docs/adapters) für deine Zielumgebung (z.B. Vercel, Netlify, Node-Server etc.).

---

Viel Spaß beim Entdecken deiner Bubble! 🎉
