// app.js
const STORE_KEY = "helios_site_v4";

// === CONFIG ===
const CODES = {
  portal: "That's_no_moon",
};

// Western sabotaged site (exact):
const TARGET = { lat: 38.313467, lon: -111.147811 };
// ==============

function loadState(){
  try {
    return JSON.parse(localStorage.getItem(STORE_KEY)) ?? {
      progress: { portalUnlocked: false },
      drama: { invalidTargetAttempts: 0, captchaFails: 0 },
      timers: { strikeWindowStartMs: null }
    };
  } catch {
    return {
      progress: { portalUnlocked: false },
      drama: { invalidTargetAttempts: 0, captchaFails: 0 },
      timers: { strikeWindowStartMs: null }
    };
  }
}
function saveState(state){ localStorage.setItem(STORE_KEY, JSON.stringify(state)); }
function resetAll(){ localStorage.removeItem(STORE_KEY); }
function getProgress(){ return loadState().progress; }

function normalize(s){ return (s ?? "").trim(); }
function checkCode(lockId, input){ return normalize(input) === normalize(CODES[lockId]); }

function unlock(lockId){
  const s = loadState();
  if (lockId === "portal") s.progress.portalUnlocked = true;
  saveState(s);
}

function requireUnlockOrRedirect(lockId, redirectTo){
  const p = getProgress();
  const ok = (lockId === "portal" && p.portalUnlocked);
  if (!ok) window.location.href = redirectTo;
}

// ---- Drama counters ----
function incrementInvalidTargetAttempts(){
  const s = loadState();
  s.drama.invalidTargetAttempts = (s.drama.invalidTargetAttempts ?? 0) + 1;
  saveState(s);
  return s.drama.invalidTargetAttempts;
}
function resetInvalidTargetAttempts(){
  const s = loadState();
  s.drama.invalidTargetAttempts = 0;
  saveState(s);
}
function getCaptchaFails(){
  return loadState().drama.captchaFails ?? 0;
}
function noteCaptchaFail(){
  const s = loadState();
  s.drama.captchaFails = (s.drama.captchaFails ?? 0) + 1;
  saveState(s);
}
function resetCaptchaFails(){
  const s = loadState();
  s.drama.captchaFails = 0;
  saveState(s);
}

// ---- Decimal parsing + exact match (canonicalized) ----
function parseDecimal(s){
  if (s === null || s === undefined) return null;
  const t = String(s).trim();
  if (!t) return null;
  if (!/^[+-]?\d+(\.\d+)?$/.test(t)) return null;
  const v = Number(t);
  if (!Number.isFinite(v)) return null;
  // optional sanity bounds
  return v;
}
function canonical(n){
  // 6 decimals ensures "38.3134670" matches "38.313467"
  return Number(n).toFixed(6);
}
function canonicalPair(lat, lon){
  return `${canonical(lat)},${canonical(lon)}`;
}

// ---- Cosmetic countdown ----
function startCosmeticCountdown(totalSeconds, el){
  const s = loadState();

  // Keep a consistent countdown within the browser session (reload-friendly)
  if (!s.timers.strikeWindowStartMs){
    s.timers.strikeWindowStartMs = Date.now();
    saveState(s);
  }

  function render(){
    const state = loadState();
    const start = state.timers.strikeWindowStartMs ?? Date.now();
    const elapsed = Math.floor((Date.now() - start) / 1000);
    const remaining = Math.max(0, totalSeconds - elapsed);

    const mm = String(Math.floor(remaining / 60)).padStart(2, "0");
    const ss = String(remaining % 60).padStart(2, "0");
    el.textContent = `${mm}:${ss}`;
  }

  render();
  setInterval(render, 1000);
}
