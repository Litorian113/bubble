<script lang="ts">
import data from '$lib/data.json';
import { onMount, onDestroy } from 'svelte';

interface StateType {
  counts: {[key: string]: number};
  lastCategory: string | null;
  round: number;
}

// Partikel-Interface für die Effekte
interface Particle {
  x: number;
  y: number;
  size: number;
  speedX: number;
  speedY: number;
  rotation: number;
  rotationSpeed: number;
  opacity: number;
  symbol: string;
  color: string;
  type: 'leaf' | 'binary' | 'coin';
}

let particles: Particle[] = [];
let canvas: HTMLCanvasElement;
let animationFrameId: number;
let finalBubble: string = ''; // Deklariere finalBubble hier

const state: StateType = {
  counts: { technologia: 0, climate: 0, culture: 0, economy: 0 },
  lastCategory: null,          // bubble the user is currently drifting toward
  round: 0                     // increments after every choice
};

// Array für die aktuellen Auswahlmöglichkeiten
let currentChoices: string[] = [];

// Status des Spiels
let gameActive = true;
let finalResult = '';

/** Figure position on the rail: 0–6  →  use to drive your motor/servo */
function figureStep(count: number): number {
  return Math.min(Math.max(count, 0), 6);
}

/** Pick n random words from a given array, excluding ones already on screen */
function randomSample(array: string[], n: number, exclude: string[] = []): string[] {
  const pool = array.filter(w => !exclude.includes(w));
  return Array.from({ length: n }, () =>
    pool.splice(Math.floor(Math.random() * pool.length), 1)[0]
  );
}

/**
 * Returns an array of 4 words for the next round
 * and remembers how many of them come from the current bubble.
 */
function getNextChoices(gameData: any, gameState: StateType): string[] {
  const allCats = gameData.categories;
  const lastCatName = gameState.lastCategory;

  // ─── 1. Determine bubble pressure ────────────────────────────────
  let bubbleWordsNeeded = 0;                    // none in round 0
  if (lastCatName) {
    const c = gameState.counts[lastCatName];       // 1..6
    // Rule: after first pick we want 2, then 3 bubble words
    if (c === 1) bubbleWordsNeeded = 2;
    else if (c >= 2) bubbleWordsNeeded = 3;
  }

  // ─── 2. Select bubble words (if any) ─────────────────────────────
  let bubbleWords: string[] = [];
  if (bubbleWordsNeeded > 0) {
    const catObj = allCats.find((c: any) => c.name === lastCatName);
    bubbleWords = randomSample(catObj.words, bubbleWordsNeeded);
  }

  // ─── 3. Fill the rest with words from *other* categories ─────────
  const nonBubbleWordsNeeded = 4 - bubbleWords.length;
  let otherWords: string[] = [];
  if (nonBubbleWordsNeeded > 0) {
    const others = allCats
      .filter((c: any) => c.name !== lastCatName)
      .flatMap((c: any) => c.words);
    otherWords = randomSample(others, nonBubbleWordsNeeded);
  }

  // Mix and return
  const choices = [...bubbleWords, ...otherWords]
    .sort(() => Math.random() - 0.5);

  return choices;
}

/**
 * wordClicked – the label on the button the user just pressed
 *             – look up its category, update counts, move figure, etc.
 */
function onChoice(wordClicked: string): void {
  // 0. Find this word's category
  const catObj = data.categories.find((cat: any) =>
    cat.words.includes(wordClicked)
  );
  
  if (!catObj) {
    console.error('Kategorie nicht gefunden für:', wordClicked);
    return;
  }
  
  const catName = catObj.name;

  // 1. Update counts
  if (state.lastCategory === catName) {
    // user stayed inside the bubble
    state.counts[catName] += 1;
  } else {
    // user switched away – pull old bubble back by 1 step
    if (state.lastCategory) {
      state.counts[state.lastCategory] =
        Math.max(state.counts[state.lastCategory] - 1, 0);
    }
    // and start drifting toward the new bubble
    state.counts[catName] += 1;
    state.lastCategory = catName;
  }

  // 2. Cap counts at 6
  Object.keys(state.counts).forEach(
    k => (state.counts[k] = Math.min(state.counts[k], 6))
  );

  // 3. Move figure – you'll map this integer to motor steps / DMX
  const step = figureStep(state.counts[catName]);
  // moveFigureTo(step); // Auskommentiert, da es keine eigene Implementierung ist

  // 4. Prepare next round of buttons
  state.round += 1;
  
  // Nach 10 Runden beenden wir das Spiel
  if (state.round >= 10) {
    endGame();
  } else {
    currentChoices = getNextChoices(data, state);
  }
}

// Funktion zum Beenden des Spiels
function endGame(): void {
  gameActive = false;
  finalBubble = Object.entries(state.counts)
    .sort((a, b) => b[1] - a[1])[0][0];   // category with max count
  
  // Zeigt das Ergebnis an
  finalResult = getBubbleDescription(finalBubble);

  // Partikeleffekt basierend auf der finalen Bubble starten
  if (canvas) {
    particles = []; // Bestehende Partikel löschen
    let particleType: 'leaf' | 'binary' | 'coin' = 'leaf'; // Standardtyp
    if (finalBubble === 'technologia') {
      particleType = 'binary';
    } else if (finalBubble === 'economy') {
      particleType = 'coin';
    } else if (finalBubble === 'climate') {
      particleType = 'leaf';
    }
    createParticles(canvas.width / 2, canvas.height / 2, particleType, 50); // Erzeuge mehr Partikel für den Endeffekt
  }
}

// Funktion um eine Beschreibung der finalen Bubble zu erhalten
function getBubbleDescription(bubble: string): string {
  const descriptions: {[key: string]: string} = {
    technologia: "You have landed in the Technology Bubble. You are highly interested in new technologies and their impact on the future.",
    climate: "You have landed in the Climate Bubble. Environmental issues and sustainability are particularly important to you.",
    culture: "You have landed in the Culture Bubble. Art, traditions, and cultural expressions play an important role in your life.",
    economy: "You have landed in the Economy Bubble. Finance, markets, and economic relationships are topics that particularly interest you."
  };
  
  return descriptions[bubble] || "No clear bubble identified.";
}

// Starte das Spiel
function startGame(): void {
  // Zurücksetzen des Status
  state.counts = { technologia: 0, climate: 0, culture: 0, economy: 0 };
  state.lastCategory = null;
  state.round = 0;
  gameActive = true;
  finalResult = '';
  
  // Erste Auswahl generieren
  currentChoices = getNextChoices(data, state);
}

// Initialisierung des Spiels beim Laden
startGame();

// ─── PARTIKELANIMATION ────────────────────────────────────────────────
// Partikel zeichnen
function drawParticles() {
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  ctx.clearRect(0, 0, canvas.width, canvas.height);

  particles.forEach(p => {
    ctx.save();
    ctx.translate(p.x, p.y);
    ctx.rotate(p.rotation);
    ctx.globalAlpha = p.opacity;

    // Partikelsymbole je nach Typ auswählen
    ctx.beginPath();
    if (p.type === 'leaf') {
      // Einfache Blattform
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.moveTo(0, 0);
      ctx.quadraticCurveTo(p.size / 2, -p.size, p.size, 0);
      ctx.quadraticCurveTo(p.size / 2, p.size, 0, 0);
      ctx.fill();
    } else if (p.type === 'binary') {
      ctx.fillStyle = p.color;
      ctx.font = `${p.size}px 'Roboto Mono', monospace`;
      ctx.fillText(Math.random() < 0.5 ? '0' : '1', 0, 0);
    } else if (p.type === 'coin') {
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(0, 0, p.size / 2, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = '#000';
      ctx.lineWidth = 2;
      ctx.stroke();
      ctx.fillStyle = '#000';
      ctx.font = `${p.size / 2}px 'Roboto Mono', monospace`;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('$', 0, 0);
    }

    ctx.restore();
  });
}

// Partikel aktualisieren
function updateParticles() {
  particles = particles.filter(p => p.opacity > 0);

  particles.forEach(p => {
    p.x += p.speedX;
    p.y += p.speedY;
    p.rotation += p.rotationSpeed;
    p.opacity -= 0.004; // Fade out langsamer
  });
}

// Neue Partikel erzeugen
function createParticles(x: number, y: number, type: 'leaf' | 'binary' | 'coin', countOverride?: number) {
  const colors = type === 'leaf' ? ['#228B22', '#3CB371', '#8FBC8F'] : 
                 type === 'binary' ? ['#000000', '#333333'] : 
                 ['#FFD700', '#F0E68C', '#BDB76B'];
  const count = countOverride || 10 + Math.random() * 20;

  for (let i = 0; i < count; i++) {
    particles.push({
      x: Math.random() * canvas.width, // Partikel über den gesamten Bildschirm verteilen
      y: Math.random() * canvas.height,
      size: 10 + Math.random() * 20, // Angepasste Größe für bessere Sichtbarkeit
      speedX: -1 + Math.random() * 2, // Langsamere, variierende Geschwindigkeit
      speedY: -1 + Math.random() * 2,
      rotation: Math.random() * Math.PI * 2,
      rotationSpeed: -0.05 + Math.random() * 0.1,
      opacity: 0.5 + Math.random() * 0.5, // Variierende Opazität für Tiefe
      symbol: '', // Symbol wird jetzt durch den Typ bestimmt
      color: colors[Math.floor(Math.random() * colors.length)],
      type
    });
  }
}

// Animation starten
function animate() {
  updateParticles();
  drawParticles();
  animationFrameId = requestAnimationFrame(animate);
}

// Animation stoppen
function stopAnimation() {
  cancelAnimationFrame(animationFrameId);
}

// Canvas für Partikel erstellen
onMount(() => {
  const canvasEl = document.getElementById('particleCanvas') as HTMLCanvasElement;
  if (canvasEl) {
    canvas = canvasEl;
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    // Keine Partikel beim Start erzeugen, erst am Ende
    // createParticles(canvas.width / 2, canvas.height / 2, 'leaf'); 

    animate();
  }

  return () => {
    stopAnimation();
  };
});
</script>

<div class="container">
  <h1>BUBBLE EXPERIMENT</h1>
  
  {#if gameActive}
    <div class="game-status">
      <p>ROUND {state.round + 1}/10</p>
    </div>
    
    <div class="button-container">
      {#each currentChoices as word}
        <button on:click={() => onChoice(word)} class="word-button">
          {word}
        </button>
      {/each}
    </div>
    
    <div class="bubble-status">
      <h2>FILTERBUBBLE</h2>
      <div class="bubble-meters">
        {#each Object.entries(state.counts) as [category, count]}
          <div class="bubble-meter">
            <span>{category}</span>
            <div class="meter">
              <div class="meter-fill" style="width: {count * 100/6}%"></div>
            </div>
            <span>{count}/6</span>
          </div>
        {/each}
      </div>
    </div>
  {:else}
    <div class="result">
      <h2>{finalBubble.toUpperCase()} BUBBLE</h2>
      <p>{finalResult}</p>
      <button on:click={startGame} class="restart-button">
        RESTART
      </button>
    </div>
  {/if}

  <!-- Partikel-Canvas -->
  <canvas id="particleCanvas" class="particle-canvas"></canvas>
</div>


<style>
/* Swiss/Bauhaus Style in Schwarz-Weiß */
@import url('https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@400;700&display=swap');

:global(body) {
  margin: 0;
  padding: 0;
  background-color: #ffffff;
  color: #000000;
  font-family: 'Roboto Mono', monospace;
  line-height: 1.5;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 4rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
}

h1 {
  text-align: center;
  margin-bottom: 4rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 3px;
  font-size: 2.5rem;
  position: relative;
}

h1::after {
  content: "";
  position: absolute;
  left: 50%;
  bottom: -20px;
  transform: translateX(-50%);
  width: 60px;
  height: 6px;
  background-color: #000;
}

.game-status {
  text-align: center;
  margin-bottom: 2rem;
  font-weight: bold;
}

.button-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 0rem;
  width: 100%;
  max-width: 600px;
}

.word-button {
  padding: 1.5rem;
  font-size: 1.2rem;
  border: 2px solid #000;
  background-color: #000;
  color: #fff;
  cursor: pointer;
  transition: all 0.2s ease;
  text-transform: uppercase;
  font-weight: bold;
  letter-spacing: 1px;
  position: relative;
}

.word-button:active {
  background-color: #fff;
  color: #000;
  transform: translateY(-3px);
  box-shadow: 4px 4px 0 #000;
}

.bubble-status {
  margin-top: 3rem;
  width: 100%;
  max-width: 600px;
}

h2 {
  text-transform: uppercase;
  font-weight: 700;
  font-size: 1.2rem;
  letter-spacing: 2px;
  margin-bottom: 2rem;
  text-align: center;
}

.bubble-meters {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
}

.bubble-meter {
  display: grid;
  grid-template-columns: 120px 1fr 50px;
  align-items: center;
  gap: 1rem;
}

.bubble-meter span:first-child {
  text-transform: uppercase;
  font-weight: bold;
  text-align: right;
}

.bubble-meter span:last-child {
  text-align: center;
}

.meter {
  height: 30px;
  position: relative;
  background-color: transparent;
  border: 2px solid #000;
}

.meter-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background-color: #000;
  transition: width 0.3s ease;
}

.result {
  text-align: center;
  padding: 3rem;
  border: 3px solid #000;
  margin-top: 1rem;
  max-width: 600px;
}

.restart-button {
  margin-top: 2rem;
  padding: 1rem 2rem;
  background-color: #000;
  color: white;
  border: 2px solid #000;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 1px;
  transition: all 0.3s ease;
  position: relative;
}

.restart-button:hover {
  background-color: #fff;
  color: #000;
  transform: translateY(-3px);
  box-shadow: 4px 4px 0 #000;
}

/* Partikel-Canvas */
.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
</style>