import { useState } from "react";

const GRID = 24;
const PX = 16; // px per cell in preview

// ── RNG (mirrors Solidity LCG exactly) ──
function lcg(seed) {
  const rolls = [];
  let s = (seed * 6969 + 42) >>> 0;
  for (let i = 0; i < 7; i++) {
    s = ((s * 1664525 + 1013904223) >>> 0);
    rolls.push(s);
  }
  return rolls;
}

function pick(roll, weights) {
  const r = roll % 100;
  let c = 0;
  for (let i = 0; i < weights.length; i++) {
    c += weights[i];
    if (r < c) return i;
  }
  return weights.length - 1;
}

function getTraits(seed) {
  const r = lcg(seed);
  return {
    bg:        pick(r[0], [22,20,18,15,12,8,3,2]),
    body:      pick(r[1], [28,24,18,14,10,1,5]),
    antenna:   pick(r[2], [40,30,20,10]),
    eyes:      pick(r[3], [35,28,20,12,5]),
    legs:      pick(r[4], [55,30,15]),
    shell:     pick(r[5], [35,28,20,12,5]),
    accessory: pick(r[6], [90,5,5]),
  };
}

// ── COLORS ──
const BODY_COLOR = ["#6b4423","#2e1a0e","#d4b896","#3a5e28","#1e3a5e","#5e1e1e","#8a6a00"];
const BODY_SHADE = ["#3d2610","#1a0e06","#a08060","#223818","#122238","#381212","#5c4400"];
const BG_COLOR   = ["#080808","#0f1a0f","#1a1208","#1a0c08","#080f1a","#06000f","#1a1400","#000f08"];
const BG_NAMES   = ["Void","Sewer","Dirt","Rust","Lab","Neon Night","Gold","Glitch"];
const BODY_NAMES = ["Brown","Dark","Albino","Green","Blue","Red","Gold"];
const ANT_NAMES  = ["Straight","Curly","Broken","Long"];
const EYE_NAMES  = ["Beady","Big","Angry","Dead","Laser"];
const LEG_NAMES  = ["6 Legs","4 Legs","8 Legs"];
const SHELL_NAMES= ["Smooth","Ridged","Spotted","Cracked","Glowing"];
const ACC_NAMES  = ["None","Wings","Crown"];

// ── PIXEL CANVAS ──
// Returns a 24×24 array of color strings (null = transparent)
function buildPixels(t) {
  const grid = Array.from({length: GRID}, () => Array(GRID).fill(null));

  const bc = BODY_COLOR[t.body];
  const bs = BODY_SHADE[t.body];

  function row(c1, c2, r, color) {
    for (let c = c1; c <= c2; c++) grid[r][c] = color;
  }
  function p(c, r, color) {
    if (r >= 0 && r < GRID && c >= 0 && c < GRID) grid[r][c] = color;
  }

  // Background
  const bgc = BG_COLOR[t.bg];
  for (let r = 0; r < GRID; r++) for (let c = 0; c < GRID; c++) grid[r][c] = bgc;

  // BG textures
  if (t.bg === 1) { // Sewer drips
    [[3,0],[3,1],[3,2],[9,0],[9,1],[16,0],[16,1],[16,2],[16,3],[21,0],[21,1]].forEach(([c,r]) => p(c,r,"#1a3a1a"));
  }
  if (t.bg === 3) { // Rust streaks
    [5,12,18].forEach(r => row(0,23,r,"#2a1008"));
  }
  if (t.bg === 5) { // Neon grid dots
    for (let r = 0; r < 24; r+=4) for (let c = 0; c < 24; c+=4) p(c,r,"#0a1a00");
  }
  if (t.bg === 6) { // Gold diagonal
    for (let i = 0; i < 24; i++) p(i,i,"#2a2000");
  }
  if (t.bg === 7) { // Glitch tears
    row(0,23,4,"#001a0a"); row(0,15,11,"#001a0a");
    row(5,23,16,"#1a0005"); row(0,23,20,"#001a0a");
  }

  // Wings (behind body)
  if (t.accessory === 1) {
    const wL = [[6,9],[5,9],[4,9],[6,10],[5,10],[4,10],[3,10],[6,11],[5,11],[4,11],[3,11],[2,11],[6,12],[5,12],[4,12],[3,12],[6,13],[5,13]];
    const wR = [[17,9],[18,9],[19,9],[17,10],[18,10],[19,10],[20,10],[17,11],[18,11],[19,11],[20,11],[21,11],[17,12],[18,12],[19,12],[20,12],[17,13],[18,13]];
    const wc = [[5,10],[4,11],[5,11],[18,10],[19,11],[18,11]]; // highlights
    wL.forEach(([c,r]) => p(c,r,"#a0cc44"));
    wR.forEach(([c,r]) => p(c,r,"#a0cc44"));
    wc.forEach(([c,r]) => p(c,r,"#b0dd55"));
    [[4,9],[3,10],[2,11],[3,12],[19,9],[20,10],[21,11],[20,12]].forEach(([c,r]) => p(c,r,"#88aa33"));
    [[6,13],[5,13],[17,13],[18,13]].forEach(([c,r]) => p(c,r,"#88aa33"));
  }

  // Legs
  const legRows6 = [10,12,14,16,18,20];
  const legRows4 = [10,13];
  const legRows8 = [9,11,13,15,16,17,18,19];
  const legSet = t.legs === 1 ? legRows4 : t.legs === 2 ? legRows8 : legRows6;
  const count = t.legs === 1 ? 2 : t.legs === 2 ? 4 : 3;
  for (let i = 0; i < count; i++) {
    const r = legSet[i];
    p(6,r,bc); p(5,r,bc); p(4,r+1,bc);
    p(17,r,bc); p(18,r,bc); p(19,r+1,bc);
  }

  // Body
  row(9,14,8,bc);
  row(8,15,9,bc);
  row(7,16,10,bc);
  for (let r = 11; r <= 18; r++) { row(7,16,r,bc); p(7,r,bs); p(16,r,bs); }
  row(8,15,19,bc);
  row(9,14,20,bc);
  row(10,13,21,bc);

  // Shell
  if (t.shell === 1) { [10,13,16,19].forEach(r => row(8,15,r,bs)); }
  if (t.shell === 2) { [[9,10],[12,11],[10,13],[14,13],[11,15],[9,17],[13,17],[11,19]].forEach(([c,r]) => p(c,r,bs)); }
  if (t.shell === 3) {
    [[11,9],[12,10],[11,11],[12,12],[11,13],[12,14],[11,15],[12,16],[11,17],[12,18],[11,19]].forEach(([c,r]) => p(c,r,bs));
  }
  if (t.shell === 4) {
    row(9,14,8,"#c8ff00"); row(9,14,21,"#c8ff00");
    [10,14,18].forEach(r => { p(7,r,"#c8ff00"); p(16,r,"#c8ff00"); });
  }

  // Head
  row(10,13,4,bc);
  row(9,14,5,bc); p(9,5,bs); p(14,5,bs);
  row(9,14,6,bc); p(9,6,bs); p(14,6,bs);
  row(9,14,7,bc);
  p(8,8,bs); p(9,8,bc); row(10,13,8,bc); p(14,8,bc); p(15,8,bs);

  // Eyes
  if (t.eyes === 0) { p(10,6,"#0a0a0a"); p(13,6,"#0a0a0a"); }
  if (t.eyes === 1) {
    p(10,5,"#fff"); p(11,5,"#fff"); p(10,6,"#0a0a0a"); p(11,6,"#fff");
    p(13,5,"#fff"); p(14,5,"#fff"); p(13,6,"#0a0a0a"); p(14,6,"#fff");
  }
  if (t.eyes === 2) { p(10,5,"#0a0a0a"); p(13,5,"#0a0a0a"); p(10,6,"#cc2200"); p(13,6,"#cc2200"); }
  if (t.eyes === 3) {
    p(10,5,"#666"); p(11,6,"#666"); p(11,5,"#666"); p(10,6,"#666");
    p(13,5,"#666"); p(14,6,"#666"); p(14,5,"#666"); p(13,6,"#666");
  }
  if (t.eyes === 4) {
    p(10,6,"#ff3300"); p(13,6,"#ff3300");
    p(5,6,"#ff3300"); p(6,6,"#ff5500"); p(17,6,"#ff5500"); p(18,6,"#ff3300");
  }

  // Antenna
  if (t.antenna === 0) { p(10,3,bs); p(10,2,bs); p(10,1,"#fff"); p(13,3,bs); p(13,2,bs); p(13,1,"#fff"); }
  if (t.antenna === 1) {
    p(10,3,bs); p(10,2,bs); p(9,2,bs); p(8,2,bs); p(8,1,"#fff");
    p(13,3,bs); p(13,2,bs); p(14,2,bs); p(15,2,bs); p(15,1,"#fff");
  }
  if (t.antenna === 2) { p(10,3,bs); p(10,2,bs); p(9,1,bs); p(13,3,bs); p(13,2,bs); p(14,1,bs); }
  if (t.antenna === 3) {
    p(10,3,bs); p(10,2,bs); p(10,1,bs); p(10,0,"#fff");
    p(13,3,bs); p(13,2,bs); p(13,1,bs); p(13,0,"#fff");
  }

  // Crown
  if (t.accessory === 2) {
    row(9,14,3,"#ffd700");
    p(9,2,"#ffd700"); p(11,1,"#ffd700"); p(12,1,"#ffd700"); p(14,2,"#ffd700");
    p(9,2,"#ff4500"); p(12,1,"#ff4500"); p(14,2,"#ff4500");
  }

  // Gold body ring
  if (t.body === 6) {
    p(6,9,"#ffd700"); p(6,14,"#ffd700"); p(6,18,"#ffd700");
    p(17,9,"#ffd700"); p(17,14,"#ffd700"); p(17,18,"#ffd700");
    row(9,14,7,"#ffd700"); row(9,14,22,"#ffd700");
  }

  return grid;
}

// ── ROACH CANVAS ──
function RoachCanvas({ traits, size = PX }) {
  const pixels = buildPixels(traits);
  const dim = GRID * size;
  return (
    <svg width={dim} height={dim} style={{ imageRendering: "pixelated", display: "block" }}>
      {pixels.map((row, r) =>
        row.map((color, c) =>
          color ? (
            <rect key={`${r}-${c}`} x={c * size} y={r * size} width={size} height={size} fill={color} />
          ) : null
        )
      )}
    </svg>
  );
}

// ── TRAIT BADGE ──
function Badge({ label, value }) {
  return (
    <div style={{ background: "#111", border: "1px solid #222", borderRadius: 4, padding: "4px 8px", fontSize: 11, color: "#888" }}>
      <span style={{ color: "#444", marginRight: 4 }}>{label}</span>
      <span style={{ color: "#c8ff00" }}>{value}</span>
    </div>
  );
}

// ── MAIN APP ──
export default function App() {
  const [seed, setSeed] = useState(1);
  const [inputVal, setInputVal] = useState("1");
  const [view, setView] = useState("single"); // single | gallery

  const traits = getTraits(seed);
  const isRare = traits.body === 6 || traits.accessory > 0;

  const gallerySeeds = Array.from({length: 12}, (_, i) => i + 1);

  return (
    <div style={{ background: "#0a0a0a", minHeight: "100vh", color: "#fff", fontFamily: "monospace", padding: 24 }}>
      {/* Header */}
      <div style={{ marginBottom: 24 }}>
        <div style={{ fontSize: 22, fontWeight: 900, letterSpacing: 2, color: "#ff4500" }}>ONCHAIN ROACHES</div>
        <div style={{ fontSize: 11, color: "#444", marginTop: 2 }}>pixel art preview — 24×24 grid</div>
      </div>

      {/* View toggle */}
      <div style={{ display: "flex", gap: 8, marginBottom: 24 }}>
        {["single","gallery"].map(v => (
          <button key={v} onClick={() => setView(v)} style={{
            background: view === v ? "#c8ff00" : "#111",
            color: view === v ? "#000" : "#666",
            border: "1px solid " + (view === v ? "#c8ff00" : "#222"),
            borderRadius: 4, padding: "6px 16px", fontFamily: "monospace",
            fontWeight: 700, fontSize: 11, cursor: "pointer", textTransform: "uppercase", letterSpacing: 1
          }}>{v}</button>
        ))}
      </div>

      {view === "single" ? (
        <div style={{ display: "flex", gap: 32, flexWrap: "wrap", alignItems: "flex-start" }}>
          {/* Canvas */}
          <div>
            <div style={{ border: "1px solid #222", display: "inline-block" }}>
              <RoachCanvas traits={traits} size={16} />
            </div>
            {isRare && (
              <div style={{ marginTop: 8, fontSize: 11, color: "#ffd700", letterSpacing: 1 }}>★ RARE</div>
            )}
          </div>

          {/* Controls + traits */}
          <div style={{ flex: 1, minWidth: 220 }}>
            <div style={{ marginBottom: 16 }}>
              <div style={{ fontSize: 11, color: "#444", marginBottom: 6 }}>TOKEN ID / SEED</div>
              <div style={{ display: "flex", gap: 8 }}>
                <input
                  type="number" min={1} max={6969}
                  value={inputVal}
                  onChange={e => setInputVal(e.target.value)}
                  onKeyDown={e => e.key === "Enter" && setSeed(Math.max(1, Math.min(6969, parseInt(inputVal) || 1)))}
                  style={{
                    background: "#111", border: "1px solid #333", color: "#fff",
                    fontFamily: "monospace", fontSize: 14, padding: "6px 10px",
                    borderRadius: 4, width: 90
                  }}
                />
                <button onClick={() => setSeed(Math.max(1, Math.min(6969, parseInt(inputVal) || 1)))} style={{
                  background: "#c8ff00", color: "#000", border: "none", borderRadius: 4,
                  fontFamily: "monospace", fontWeight: 700, fontSize: 11, padding: "6px 12px", cursor: "pointer"
                }}>RENDER</button>
                <button onClick={() => { const r = Math.floor(Math.random()*6969)+1; setSeed(r); setInputVal(String(r)); }} style={{
                  background: "#111", color: "#666", border: "1px solid #222", borderRadius: 4,
                  fontFamily: "monospace", fontSize: 11, padding: "6px 12px", cursor: "pointer"
                }}>RNG</button>
              </div>
            </div>

            {/* Trait badges */}
            <div style={{ fontSize: 11, color: "#444", marginBottom: 8 }}>TRAITS</div>
            <div style={{ display: "flex", flexWrap: "wrap", gap: 6 }}>
              <Badge label="BG"   value={BG_NAMES[traits.bg]} />
              <Badge label="BODY" value={BODY_NAMES[traits.body]} />
              <Badge label="ANT"  value={ANT_NAMES[traits.antenna]} />
              <Badge label="EYES" value={EYE_NAMES[traits.eyes]} />
              <Badge label="LEGS" value={LEG_NAMES[traits.legs]} />
              <Badge label="SHELL" value={SHELL_NAMES[traits.shell]} />
              <Badge label="ACC"  value={ACC_NAMES[traits.accessory]} />
              <Badge label="RARITY" value={isRare ? "Rare" : "Common"} />
            </div>

            {/* Nav arrows */}
            <div style={{ display: "flex", gap: 8, marginTop: 20 }}>
              <button onClick={() => { const n = Math.max(1, seed-1); setSeed(n); setInputVal(String(n)); }} style={{
                background: "#111", color: "#888", border: "1px solid #222",
                borderRadius: 4, fontFamily: "monospace", fontSize: 13, padding: "6px 14px", cursor: "pointer"
              }}>← PREV</button>
              <button onClick={() => { const n = Math.min(6969, seed+1); setSeed(n); setInputVal(String(n)); }} style={{
                background: "#111", color: "#888", border: "1px solid #222",
                borderRadius: 4, fontFamily: "monospace", fontSize: 13, padding: "6px 14px", cursor: "pointer"
              }}>NEXT →</button>
            </div>
          </div>
        </div>
      ) : (
        // Gallery view
        <div>
          <div style={{ fontSize: 11, color: "#444", marginBottom: 16 }}>TOKENS #1 – #12</div>
          <div style={{ display: "flex", flexWrap: "wrap", gap: 12 }}>
            {gallerySeeds.map(s => {
              const t = getTraits(s);
              const rare = t.body === 6 || t.accessory > 0;
              return (
                <div key={s} onClick={() => { setSeed(s); setInputVal(String(s)); setView("single"); }}
                  style={{
                    cursor: "pointer", border: "1px solid " + (rare ? "#ffd700" : "#1a1a1a"),
                    background: "#0d0d0d", padding: 8,
                    transition: "border-color 0.1s"
                  }}>
                  <RoachCanvas traits={t} size={10} />
                  <div style={{ marginTop: 6, fontSize: 10, color: "#333", textAlign: "center" }}>
                    #{s} {rare ? <span style={{ color: "#ffd700" }}>★</span> : ""}
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      )}
    </div>
  );
}
