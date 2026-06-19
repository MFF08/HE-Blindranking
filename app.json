// Utilities
function seededRandom(seed) {
  let s = 1;
  for (let i = 0; i < seed.length; i++) {
    s = (s * 31 + seed.charCodeAt(i)) % 2147483647;
  }
  return () => {
    s = (s * 48271) % 2147483647;
    return s / 2147483647;
  };
}
function seededShuffle(array, seed) {
  const rand = seededRandom(seed);
  const arr = array.slice();
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(rand() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}
const qs = s => document.querySelector(s);
const qsa = s => Array.from(document.querySelectorAll(s));

// State
const state = {
  categories: [],
  currentCategory: null,
  seed: "",
  items: [],
  index: 0,
  slots: [null, null, null, null, null],
  startTime: 0,
  finished: false
};

// Refs
const tabsWrap = qs('#categoryTabs');
const seedInput = qs('#seed');
const startBtn = qs('#startBtn');
const gameSection = qs('#gameSection');
const resultSection = qs('#resultSection');
const itemCard = qs('#itemCard');
const progressEl = qs('#progress');
const timeEl = qs('#time');
const resultList = qs('#resultList');
const shareLink = qs('#shareLink');
const copyLinkBtn = qs('#copyLink');
const playAgainBtn = qs('#playAgain');
const placeSkipBtn = qs('#placeSkip');
const resetRoundBtn = qs('#resetRound');

// Load categories and build tabs
async function loadCategories() {
  try {
    const res = await fetch('data/categories.json', {cache: 'no-store'});
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    state.categories = Array.isArray(data.categories) ? data.categories : [];

    // Build tab buttons
    tabsWrap.innerHTML = '';
    state.categories.forEach((cat, i) => {
      const btn = document.createElement('button');
      btn.className = 'tab-btn';
      btn.setAttribute('role', 'tab');
      btn.setAttribute('aria-selected', 'false');
      btn.dataset.catId = cat.id;
      btn.textContent = cat.title;
      btn.addEventListener('click', () => selectCategory(cat.id));
      tabsWrap.appendChild(btn);
    });

    // URL params
    const url = new URL(window.location.href);
    const catParam = url.searchParams.get('cat');
    const seedParam = url.searchParams.get('seed');

    if (catParam && state.categories.some(c => c.id === catParam)) {
      selectCategory(catParam);
    } else if (state.categories[0]) {
      // optional: erste Kategorie vorwählen
      selectCategory(state.categories[0].id);
    }
    if (seedParam) seedInput.value = seedParam;
  } catch (e) {
    console.error(e);
    alert('Kategorien konnten nicht geladen werden. Prüfe data/categories.json.');
  }
}

function selectCategory(catId) {
  const cat = state.categories.find(c => c.id === catId);
  if (!cat) return;
  state.currentCategory = cat;

  // Visual active
  qsa('.tab-btn').forEach(b => b.setAttribute('aria-selected', String(b.dataset.catId === catId)));

  // Update share link if seed present
  if (state.seed) updateShareLink();
}

function startRound() {
  if (!state.currentCategory) {
    alert('Bitte zuerst eine Kategorie auswählen.');
    return;
  }
  const seed = seedInput.value.trim() || (state.currentCategory.id + '-' + Date.now());
  state.seed = seed;

  const shuffled = seededShuffle(state.currentCategory.items, seed);
  state.items = shuffled.slice(0, 5);
  state.index = 0;
  state.slots = [null, null, null, null, null];
  state.finished = false;
  state.startTime = performance.now();

  itemCard.textContent = 'Bereit…';
  qsa('.slot-item').forEach(el => el.textContent = '');
  updateProgress();
  gameSection.classList.remove('hidden');
  resultSection.classList.add('hidden');

  showNextItem();
  updateShareLink();
}

function updateShareLink() {
  const url = new URL(window.location.href);
  url.searchParams.set('cat', state.currentCategory.id);
  url.searchParams.set('seed', state.seed);
  shareLink.value = url.toString();
}

function updateProgress() {
  progressEl.textContent = `Item ${Math.min(state.index, 5)}/5`;
  if (!state.finished) {
    const elapsed = (performance.now() - state.startTime) / 1000;
    timeEl.textContent = `Zeit: ${elapsed.toFixed(1)}s`;
  }
}

function showNextItem() {
  if (state.index >= state.items.length) {
    finishRound();
    return;
  }
  const item = state.items[state.index];
  itemCard.textContent = item.label;
  itemCard.dataset.key = item.key;
  updateProgress();
}

function placeInSlot(slotIndex) {
  if (state.finished) return;
  if (state.index >= state.items.length) return;

  const item = state.items[state.index];
  const currentPos = state.slots.findIndex(x => x && x.key === item.key);

  if (currentPos === slotIndex) {
    // unchanged
  } else if (state.slots[slotIndex] == null) {
    state.slots[slotIndex] = item;
    document.querySelectorAll('.slot-item')[slotIndex].textContent = item.label;
    state.index++;
  } else {
    const temp = state.slots[slotIndex];
    state.slots[slotIndex] = item;
    document.querySelectorAll('.slot-item')[slotIndex].textContent = item.label;

    if (currentPos >= 0) {
      state.slots[currentPos] = temp;
      document.querySelectorAll('.slot-item')[currentPos].textContent = temp.label;
      state.index++;
    } else {
      state.items[state.index] = temp;
    }
  }

  if (state.index < 5) {
    showNextItem();
  } else {
    finishRound();
  }
}

function skipPlacement() {
  const item = state.items[state.index];
  const freeIndex = state.slots.findIndex(x => x == null);
  if (freeIndex >= 0) {
    state.slots[freeIndex] = item;
    document.querySelectorAll('.slot-item')[freeIndex].textContent = item.label;
    state.index++;
    if (state.index < 5) showNextItem(); else finishRound();
  } else {
    placeInSlot(4);
  }
}

function finishRound() {
  state.finished = true;
  const elapsed = (performance.now() - state.startTime) / 1000;
  timeEl.textContent = `Zeit: ${elapsed.toFixed(1)}s`;
  renderResults();
  gameSection.classList.add('hidden');
  resultSection.classList.remove('hidden');
}

function renderResults() {
  resultList.innerHTML = '';
  state.slots.forEach((item, idx) => {
    const li = document.createElement('li');
    li.textContent = item ? `${idx + 1}. ${item.label}` : `${idx + 1}. –`;
    resultList.appendChild(li);
  });
}

// Events
startBtn.addEventListener('click', startRound);
playAgainBtn.addEventListener('click', () => startRound());
copyLinkBtn.addEventListener('click', async () => {
  shareLink.select();
  try {
    await navigator.clipboard.writeText(shareLink.value);
    copyLinkBtn.textContent = 'Kopiert!';
    setTimeout(() => copyLinkBtn.textContent = 'Link kopieren', 1500);
  } catch {
    document.execCommand('copy');
  }
});
placeSkipBtn.addEventListener('click', skipPlacement);
resetRoundBtn.addEventListener('click', startRound);
document.querySelectorAll('.slot').forEach(el => {
  el.addEventListener('click', () => placeInSlot(Number(el.dataset.index)));
  el.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      placeInSlot(Number(el.dataset.index));
    }
  });
});

// Init
loadCategories();

