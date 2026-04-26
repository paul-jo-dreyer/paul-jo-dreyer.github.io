---
title:  Your Life in a Single Day
date:   2026-04-26 00:01:00 +0300
image: '/images/sun_dial/sun_dial.jpg'
description: A fun perspective to view your life from.
tags: ["Food for thought"]
---

----

*"Suppose you map your life, where you're at right now in time, to a single day. What's happening in your life's day?"*

----

I love when a simple idea catches me off guard. At the time of writing this, I'm almost 31, and I suppose if I had to guess how long I have here, I'd bet I'm going out when I'm 75. From this perspective, it's about 09:54 AM in the single day of my life.

Somewhere between my first and second coffee. 

I made a simple calulator to make this mapping for you. I encourage you, after mulling over your own time, see what time it would be for your parents. I think I need to spend more time with mine. 



# Try it

<style>
  .life-clock-app {
    margin: 1.5em 0;
    padding: 1.5em;
    border: 1px solid #444;
    border-radius: 8px;
    background-color: #1a1a1a;
    color: #ddd;
    font-family: 'Inter', sans-serif;
  }
  .life-clock-app * { box-sizing: border-box; }

  .lc-inputs {
    display: flex;
    flex-wrap: wrap;
    gap: 2em;
    justify-content: center;
    margin-bottom: 1.5em;
  }
  .lc-input-block {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .lc-label {
    font-size: 0.85em;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: #888;
    margin-bottom: 0.5em;
  }
  .lc-date-row {
    display: flex;
    gap: 0.5em;
  }
  .lc-select, .lc-typed-input {
    background: #222;
    color: #ddd;
    border: 1px solid #444;
    border-radius: 4px;
    padding: 0.4em 0.6em;
    font-family: inherit;
    font-size: 0.95em;
  }
  .lc-typed-input {
    margin-top: 0.5em;
    width: 6em;
    text-align: center;
  }

  /* wheel */
  .lc-wheel-frame {
    position: relative;
    width: 100px;
    height: 150px;
    background: #222;
    border: 1px solid #444;
    border-radius: 6px;
    overflow: hidden;
  }
  .lc-wheel-list {
    list-style: none;
    margin: 0;
    padding: 60px 0;
    height: 150px;
    overflow-y: scroll;
    scroll-snap-type: y mandatory;
    scrollbar-width: none;
  }
  .lc-wheel-list::-webkit-scrollbar { display: none; }
  .lc-wheel-list li {
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    scroll-snap-align: center;
    font-size: 16px;
    color: #666;
    transition: color 0.15s, font-size 0.15s;
    cursor: pointer;
  }
  .lc-wheel-list li.lc-selected {
    color: #fff;
    font-size: 22px;
    font-weight: 600;
  }
  .lc-wheel-shade {
    position: absolute;
    left: 0; right: 0;
    height: 60px;
    pointer-events: none;
    z-index: 2;
  }
  .lc-wheel-shade-top {
    top: 0;
    background: linear-gradient(to bottom, #222 0%, rgba(34,34,34,0) 100%);
  }
  .lc-wheel-shade-bottom {
    bottom: 0;
    background: linear-gradient(to top, #222 0%, rgba(34,34,34,0) 100%);
  }
  .lc-wheel-line {
    position: absolute;
    left: 0; right: 0;
    height: 1px;
    background: #555;
    pointer-events: none;
    z-index: 3;
  }
  .lc-wheel-line-top { top: 60px; }
  .lc-wheel-line-bottom { top: 90px; }

  /* clock display */
  .life-clock-app {
    --digit-w: 50px;
    --digit-h: 90px;
    --seg-thick: 6px;
    --seg-len: 36px;
    --seg-edge: 2px;
    --colon-w: 14px;
    --clock-gap: 6px;
  }
  .lc-clock {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: var(--clock-gap);
    padding: 1em 0.5em;
    background: #0a0a0a;
    border-radius: 6px;
    border: 1px solid #333;
    overflow: hidden;
  }
  .lc-digit {
    position: relative;
    width: var(--digit-w);
    height: var(--digit-h);
    flex-shrink: 0;
  }
  .lc-segment {
    position: absolute;
    background: #151025;
    transition: background 0.05s, box-shadow 0.05s;
  }
  .lc-segment.lc-on {
    background: #ae00ff;
    box-shadow: 0 0 8px #ae00ff, 0 0 14px rgba(59, 0, 87, 0.15);
  }
  .lc-seg-h {
    width: var(--seg-len);
    height: var(--seg-thick);
    left: calc((var(--digit-w) - var(--seg-len)) / 2);
    clip-path: polygon(3px 0, calc(100% - 3px) 0, 100% 50%, calc(100% - 3px) 100%, 3px 100%, 0 50%);
  }
  .lc-seg-v {
    width: var(--seg-thick);
    height: var(--seg-len);
    clip-path: polygon(0 3px, 50% 0, 100% 3px, 100% calc(100% - 3px), 50% 100%, 0 calc(100% - 3px));
  }
  .lc-seg-a { top: var(--seg-edge); }
  .lc-seg-g { top: calc((var(--digit-h) - var(--seg-thick)) / 2); }
  .lc-seg-d { bottom: var(--seg-edge); }
  .lc-seg-b { top: var(--seg-thick); right: var(--seg-edge); }
  .lc-seg-c { bottom: var(--seg-thick); right: var(--seg-edge); }
  .lc-seg-f { top: var(--seg-thick); left: var(--seg-edge); }
  .lc-seg-e { bottom: var(--seg-thick); left: var(--seg-edge); }
  .lc-colon {
    width: var(--colon-w);
    height: var(--digit-h);
    position: relative;
    flex-shrink: 0;
  }
  .lc-colon-dot {
    position: absolute;
    left: calc((var(--colon-w) - var(--seg-thick)) / 2);
    width: var(--seg-thick);
    height: var(--seg-thick);
    border-radius: 50%;
    background: #7700ff;
    box-shadow: 0 0 8px #8400ff;
  }
  .lc-colon-dot.lc-top { top: calc(var(--digit-h) * 0.31); }
  .lc-colon-dot.lc-bot { bottom: calc(var(--digit-h) * 0.31); }

  .lc-ampm {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    height: var(--digit-h);
    margin-left: 4px;
    font-family: 'Courier New', monospace;
    font-size: calc(var(--digit-h) * 0.18);
    font-weight: 700;
    letter-spacing: 0.05em;
    flex-shrink: 0;
  }
  .lc-ampm-label {
    color: #151025;
    transition: color 0.05s, text-shadow 0.05s;
  }
  .lc-ampm-label.lc-on {
    color: #ae00ff;
    text-shadow: 0 0 6px #ae00ff;
  }

  .lc-mode-toggle {
    display: flex;
    margin: 0 auto 0.75em;
    background: #222;
    border: 1px solid #444;
    border-radius: 999px;
    padding: 2px;
    width: fit-content;
  }
  .lc-mode-opt {
    background: transparent;
    border: none;
    color: #888;
    padding: 0.3em 0.9em;
    border-radius: 999px;
    cursor: pointer;
    font-family: inherit;
    font-size: 0.85em;
    letter-spacing: 0.05em;
    transition: background 0.15s, color 0.15s, box-shadow 0.15s;
  }
  .lc-mode-opt.lc-active {
    background: #ae00ff;
    color: #fff;
    box-shadow: 0 0 8px rgba(174, 0, 255, 0.5);
  }

  .lc-error {
    color: #ff8866;
    text-align: center;
    margin-top: 0.75em;
    font-size: 0.9em;
    min-height: 1.2em;
  }

  @media (max-width: 540px) {
    .life-clock-app {
      margin: 1em 0;
      padding: 0.9em 0.75em 1em;
      --digit-w: 32px;
      --digit-h: 58px;
      --seg-thick: 4px;
      --seg-len: 22px;
      --seg-edge: 2px;
      --colon-w: 10px;
      --clock-gap: 3px;
    }
    .lc-inputs {
      gap: 1em;
      margin-bottom: 1em;
      align-items: flex-start;
    }
    .lc-label {
      font-size: 0.75em;
      margin-bottom: 0.35em;
    }
    .lc-date-row {
      gap: 0.3em;
    }
    .lc-select, .lc-typed-input {
      padding: 0.35em 0.4em;
      font-size: 0.85em;
    }
    .lc-typed-input {
      width: 5em;
      margin-top: 0.4em;
    }
    .lc-wheel-frame {
      width: 78px;
      height: 110px;
    }
    .lc-wheel-list {
      height: 110px;
      padding: 44px 0;
    }
    .lc-wheel-list li {
      height: 22px;
      font-size: 14px;
    }
    .lc-wheel-list li.lc-selected {
      font-size: 18px;
    }
    .lc-wheel-shade { height: 44px; }
    .lc-wheel-line-top { top: 44px; }
    .lc-wheel-line-bottom { top: 66px; }
    .lc-mode-toggle {
      margin-bottom: 0.5em;
      padding: 1px;
    }
    .lc-mode-opt {
      padding: 0.2em 0.7em;
      font-size: 0.75em;
    }
    .lc-clock {
      padding: 0.55em 0.4em;
    }
    .lc-ampm {
      margin-left: 3px;
      font-size: calc(var(--digit-h) * 0.2);
    }
    .lc-error {
      font-size: 0.8em;
      margin-top: 0.5em;
    }
  }
  @media (max-width: 380px) {
    .life-clock-app {
      --digit-w: 26px;
      --digit-h: 48px;
      --seg-thick: 3px;
      --seg-len: 18px;
      --colon-w: 8px;
      --clock-gap: 2px;
    }
    .lc-inputs {
      gap: 0.75em;
    }
    .lc-date-row {
      flex-wrap: wrap;
      justify-content: center;
    }
    .lc-select, .lc-typed-input {
      font-size: 0.8em;
    }
    .lc-wheel-frame {
      width: 70px;
      height: 88px;
    }
    .lc-wheel-list {
      height: 88px;
      padding: 33px 0;
    }
    .lc-wheel-shade { height: 33px; }
    .lc-wheel-line-top { top: 33px; }
    .lc-wheel-line-bottom { top: 55px; }
    .lc-mode-opt {
      padding: 0.18em 0.6em;
      font-size: 0.7em;
    }
    .lc-ampm {
      font-size: 11px;
      margin-left: 2px;
    }
  }
</style>

<div class="life-clock-app" id="life-clock-app">
  <div class="lc-inputs">
    <div class="lc-input-block">
      <span class="lc-label">Birth Date</span>
      <div class="lc-date-row">
        <select class="lc-select" id="lc-month" aria-label="Month"></select>
        <select class="lc-select" id="lc-day" aria-label="Day"></select>
        <select class="lc-select" id="lc-year" aria-label="Year"></select>
      </div>
    </div>
    <div class="lc-input-block">
      <span class="lc-label">Lifespan (yrs)</span>
      <div class="lc-wheel-frame">
        <div class="lc-wheel-shade lc-wheel-shade-top"></div>
        <div class="lc-wheel-shade lc-wheel-shade-bottom"></div>
        <div class="lc-wheel-line lc-wheel-line-top"></div>
        <div class="lc-wheel-line lc-wheel-line-bottom"></div>
        <ul class="lc-wheel-list" id="lc-wheel"></ul>
      </div>
      <input type="number" class="lc-typed-input" id="lc-lifespan-input" min="1" max="150" value="80">
    </div>
  </div>

  <div class="lc-mode-toggle" id="lc-mode-toggle">
    <button type="button" class="lc-mode-opt lc-active" data-mode="12">12H</button>
    <button type="button" class="lc-mode-opt" data-mode="24">24H</button>
  </div>

  <div class="lc-clock" id="lc-clock"></div>
  <div class="lc-error" id="lc-error"></div>
</div>

<script>
(function() {
  const MONTHS = ['January','February','March','April','May','June',
                  'July','August','September','October','November','December'];
  const SEGMENTS = {
    '0': ['a','b','c','d','e','f'],
    '1': ['b','c'],
    '2': ['a','b','d','e','g'],
    '3': ['a','b','c','d','g'],
    '4': ['b','c','f','g'],
    '5': ['a','c','d','f','g'],
    '6': ['a','c','d','e','f','g'],
    '7': ['a','b','c'],
    '8': ['a','b','c','d','e','f','g'],
    '9': ['a','b','c','d','f','g']
  };
  const HORIZONTAL = new Set(['a','d','g']);

  const monthSel = document.getElementById('lc-month');
  const daySel   = document.getElementById('lc-day');
  const yearSel  = document.getElementById('lc-year');
  const wheel    = document.getElementById('lc-wheel');
  const lifeIn   = document.getElementById('lc-lifespan-input');
  const clockEl  = document.getElementById('lc-clock');
  const errorEl  = document.getElementById('lc-error');

  // Populate month dropdown
  MONTHS.forEach((name, i) => {
    const opt = document.createElement('option');
    opt.value = i + 1;
    opt.textContent = name;
    monthSel.appendChild(opt);
  });

  // Populate day dropdown
  for (let d = 1; d <= 31; d++) {
    const opt = document.createElement('option');
    opt.value = d;
    opt.textContent = d;
    daySel.appendChild(opt);
  }

  // Populate year dropdown
  const thisYear = new Date().getFullYear();
  for (let y = thisYear; y >= 1900; y--) {
    const opt = document.createElement('option');
    opt.value = y;
    opt.textContent = y;
    yearSel.appendChild(opt);
  }

  // Defaults: Spongebob premier
  monthSel.value = 5;
  daySel.value = 1;
  yearSel.value = 1999;

  // Populate wheel
  const WHEEL_MIN = 1, WHEEL_MAX = 150;
  for (let v = WHEEL_MIN; v <= WHEEL_MAX; v++) {
    const li = document.createElement('li');
    li.textContent = v;
    li.dataset.value = v;
    li.addEventListener('click', () => scrollWheelTo(v, true));
    wheel.appendChild(li);
  }

  // Build clock digits in DOM
  function makeDigit() {
    const wrap = document.createElement('div');
    wrap.className = 'lc-digit';
    ['a','b','c','d','e','f','g'].forEach(seg => {
      const s = document.createElement('div');
      s.className = `lc-segment lc-seg-${seg} ${HORIZONTAL.has(seg) ? 'lc-seg-h' : 'lc-seg-v'}`;
      s.dataset.seg = seg;
      wrap.appendChild(s);
    });
    return wrap;
  }
  function makeColon() {
    const wrap = document.createElement('div');
    wrap.className = 'lc-colon';
    const top = document.createElement('div');
    top.className = 'lc-colon-dot lc-top';
    const bot = document.createElement('div');
    bot.className = 'lc-colon-dot lc-bot';
    wrap.appendChild(top); wrap.appendChild(bot);
    return wrap;
  }
  const digits = [];
  for (let i = 0; i < 6; i++) {
    const d = makeDigit();
    digits.push(d);
    clockEl.appendChild(d);
    if (i === 1 || i === 3) clockEl.appendChild(makeColon());
  }

  const ampmEl = document.createElement('div');
  ampmEl.className = 'lc-ampm';
  const amEl = document.createElement('div');
  amEl.className = 'lc-ampm-label';
  amEl.textContent = 'AM';
  const pmEl = document.createElement('div');
  pmEl.className = 'lc-ampm-label';
  pmEl.textContent = 'PM';
  ampmEl.appendChild(amEl);
  ampmEl.appendChild(pmEl);
  clockEl.appendChild(ampmEl);

  let mode24h = false;
  document.querySelectorAll('#lc-mode-toggle .lc-mode-opt').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('#lc-mode-toggle .lc-mode-opt').forEach(b => b.classList.remove('lc-active'));
      btn.classList.add('lc-active');
      mode24h = btn.dataset.mode === '24';
      update();
    });
  });

  function setDigit(digitEl, char) {
    const on = SEGMENTS[char] || [];
    digitEl.querySelectorAll('.lc-segment').forEach(s => {
      s.classList.toggle('lc-on', on.includes(s.dataset.seg));
    });
  }
  function renderClock(h, m, s) {
    let displayH = h;
    let period = null;
    if (!mode24h) {
      period = h < 12 ? 'AM' : 'PM';
      displayH = h % 12;
      if (displayH === 0) displayH = 12;
    }
    const hStr = (!mode24h && displayH < 10)
      ? ' ' + displayH
      : String(displayH).padStart(2, '0');
    const str = hStr + String(m).padStart(2,'0') + String(s).padStart(2,'0');
    for (let i = 0; i < 6; i++) setDigit(digits[i], str[i]);
    amEl.classList.toggle('lc-on', period === 'AM');
    pmEl.classList.toggle('lc-on', period === 'PM');
    ampmEl.style.display = mode24h ? 'none' : 'flex';
  }

  // Wheel logic
  let lifespan = 80;
  let scrollTimeout = null;
  function scrollWheelTo(value, smooth) {
    const li = wheel.querySelector(`li[data-value="${value}"]`);
    if (!li) return;
    const target = li.offsetTop - (wheel.clientHeight / 2 - li.clientHeight / 2);
    wheel.scrollTo({ top: target, behavior: smooth ? 'smooth' : 'auto' });
  }
  function updateWheelHighlight() {
    const center = wheel.scrollTop + wheel.clientHeight / 2;
    let closest = null, closestDist = Infinity;
    wheel.querySelectorAll('li').forEach(li => {
      const liCenter = li.offsetTop + li.clientHeight / 2;
      const dist = Math.abs(liCenter - center);
      if (dist < closestDist) { closestDist = dist; closest = li; }
    });
    wheel.querySelectorAll('li.lc-selected').forEach(li => li.classList.remove('lc-selected'));
    if (closest) {
      closest.classList.add('lc-selected');
      const v = parseInt(closest.dataset.value, 10);
      if (v !== lifespan) {
        lifespan = v;
        lifeIn.value = v;
        update();
      }
    }
  }
  wheel.addEventListener('scroll', () => {
    if (scrollTimeout) clearTimeout(scrollTimeout);
    scrollTimeout = setTimeout(updateWheelHighlight, 50);
  });
  lifeIn.addEventListener('change', () => {
    let v = parseInt(lifeIn.value, 10);
    if (isNaN(v)) return;
    v = Math.max(WHEEL_MIN, Math.min(WHEEL_MAX, v));
    lifeIn.value = v;
    lifespan = v;
    scrollWheelTo(v, true);
    update();
  });

  // Date inputs
  [monthSel, daySel, yearSel].forEach(el => el.addEventListener('change', update));

  function update() {
    const month = parseInt(monthSel.value, 10);
    const day   = parseInt(daySel.value, 10);
    const year  = parseInt(yearSel.value, 10);
    const birth = new Date(year, month - 1, day);

    if (birth.getFullYear() !== year || birth.getMonth() !== month - 1 || birth.getDate() !== day) {
      errorEl.textContent = 'Invalid date — please check the day for that month.';
      renderClock(0,0,0);
      return;
    }
    const now = new Date();
    if (birth > now) {
      errorEl.textContent = 'Birth date is in the future.';
      renderClock(0,0,0);
      return;
    }
    if (lifespan <= 0) {
      errorEl.textContent = 'Lifespan must be positive.';
      renderClock(0,0,0);
      return;
    }
    errorEl.textContent = '';

    const ageMs = now - birth;
    const lifespanMs = lifespan * 365.25 * 24 * 3600 * 1000;
    let frac = ageMs / lifespanMs;
    if (frac >= 1) frac = 0.99999999;
    const dayMs = frac * 86400000;
    const total = Math.floor(dayMs / 1000);
    const h = Math.floor(total / 3600) % 24;
    const m = Math.floor(total / 60) % 60;
    const s = total % 60;
    renderClock(h, m, s);
  }

  // Initial state
  scrollWheelTo(lifespan, false);
  // Wait one frame for layout, then highlight & update
  requestAnimationFrame(() => {
    updateWheelHighlight();
    update();
  });

  // Tick once per real second so display stays current as time passes
  setInterval(update, 1000);
})();
</script>

