<?php
$id = $_GET['id'] ?? '';
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Sev's Blobtown Menu</title>
<style>
    /* global */
    html, body {
        margin: 0;
        padding: 0;
        height: 100%;
        overflow: hidden;
        font-family: Arial, sans-serif; /* explicit Arial */
        background: #ffffff;
        color: #000; /* default text color black */
    }

    /* top bar */
    .top-bar {
        position: relative;
        background: #000000; /* solid black, no gradient */
        color: #ffffff;
        display: flex;
        justify-content: flex-start;
        align-items: center;
        padding: 12px 20px;
        height: 56px;
        box-sizing: border-box;
        border-bottom: 2px solid rgba(255,255,255,0.08);
    }

    .top-bar .clock {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        font-size: 40px;
        user-select: none;
        opacity: 0.6;
        font-family: Arial, sans-serif;
    }

    #notifMessage {
        position: absolute;
        top: 60px;
        left: 50%;
        transform: translateX(-50%);
        color: white;
        font-weight: bold;
        font-family: Arial, sans-serif;
        font-size: 16px;
        background: rgba(0,0,0,0.6);
        padding: 6px 12px;
        border-radius: 8px;
        display: none;
        pointer-events: none;
        user-select: none;
        z-index: 10000;
        white-space: nowrap;
    }

    /* Buttons (left and right) */
    .top-bar .left-buttons button,
    .top-bar .right-buttons button {
        background: #111111;      /* solid black-ish button */
        color: #ffffff;           /* white text */
        border: none;
        padding: 8px 16px;
        margin-right: 10px;
        border-radius: 6px;
        font-weight: bold;
        cursor: pointer;
        opacity: 0.95;
        position: relative;
        transition: box-shadow 0.15s ease, transform 0.08s ease;
        font-family: Arial, sans-serif; /* ensure Arial on buttons */
    }

    .top-bar .left-buttons button:hover,
    .top-bar .right-buttons button:hover {
        transform: translateY(-1px);
        box-shadow: 0 2px 6px rgba(0,0,0,0.3);
    }

    .top-bar .right-buttons {
        margin-left: auto;
        display: flex;
        align-items: center;
    }

    .neon-red {
        color: #ff0000 !important;
        opacity: 1 !important;
        box-shadow: none; /* remove neon glow since user wants no flashy gradients */
    }

    /* main embed container */
    #embedContainer {
        position: absolute;
        top: 56px;
        left: 0;
        right: 0;
        bottom: 0;
        background: #ffffff; /* default white background */
        overflow: auto;
        color: #000000;
        font-family: Arial, sans-serif;
    }

    iframe {
        width: 100%;
        height: 100%;
        border: none;
        display: block;
    }

    #embedContainer.settings {
        color: #000;
        padding: 20px;
        font-weight: bold;
        display: flex;
        flex-direction: column;
        align-items: center;
        background: #ffffff; /* keep settings background white */
    }

    /* color picker previews and reset button */
    .color-picker {
        display: flex;
        gap: 20px;
        align-items: center;
        margin-top: 10px;
    }

    .color-preview {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        border: 2px solid #000;
        margin-left: 10px;
    }

    #resetBtn {
        margin-top: 20px;
        border: none;
        padding: 10px 20px;
        border-radius: 8px;
        cursor: pointer;
        font-weight: bold;
        background: #111; /* solid */
        color: #fff;
        opacity: 0.95;
        font-family: Arial, sans-serif;
    }

    /* small UI refinements for cat bubble and profile toggle */
    #catBubble { font-family: Arial, sans-serif; color: #fff; background: rgba(0,0,0,0.6); }
    #profileIframe { background: #ffffff; } 

    /* toggle profile button — remove gradient and animation */
    #toggleBtn {
      position: fixed;
      bottom: 20px;
      right: 20px;
      width: 45px;
      height: 45px;
      border-radius: 50%;
      border: none;
      background: #111; /* solid */
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: 0 4px 12px rgba(0,0,0,0.2);
      cursor: pointer;
      z-index: 1000;
      padding: 0;
      transition: filter 0.2s ease;
      opacity: 0.9;
    }

    #toggleBtn img { width: 24px; height: 24px; filter: invert(1); } /* white icon on dark button */

</style>
</head>
<body>

<div class="top-bar" id="topBar">
    <div class="left-buttons">
        <button onclick="goMenu()">Inventory</button>
        <!-- Browser and Chat buttons removed as requested -->
    </div>
    <div class="clock" id="clock"></div>
    <div id="notifMessage"></div>
    <div class="right-buttons">
        <button onclick="Settings()">Settings</button>
        <!-- Loader and Refresh buttons removed as requested -->
    </div>
</div>

<div id="embedContainer"></div>

<img id="cat" src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif"
     style="position: absolute; left: 100px; top: 400px; height: 100px; z-index: 9999; pointer-events: none; transition: transform 0.2s;" />

<div id="catBubble"
     style="position: absolute; background: rgba(0,0,0,0.6); color: white; padding: 6px 10px; border-radius: 10px; font-size: 14px; font-family: Arial, sans-serif; pointer-events: none; display: none; z-index: 9999;">
</div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@simonwep/pickr/dist/themes/classic.min.css" />
<script src="https://cdn.jsdelivr.net/npm/@simonwep/pickr"></script>

<script>
const userId = "<?= htmlspecialchars($id) ?>";

/* defaults — now single solid colors (no gradients) */
const defaultColor1 = '#000000'; /* top bar / primary */
const defaultColor2 = '#ffffff'; /* content background */

let themeColor1 = localStorage.getItem('themeColor1') || defaultColor1;
let themeColor2 = localStorage.getItem('themeColor2') || defaultColor2;
let catEnabled = localStorage.getItem('catEnabled') !== 'false';

const topBar = document.getElementById('topBar');
const embedContainer = document.getElementById('embedContainer');
const cat = document.getElementById('cat');
const bubble = document.getElementById('catBubble');
const notifMessage = document.getElementById('notifMessage');

/* apply solid color theme (no gradients) */
function updateTheme() {
    topBar.style.background = themeColor1;               // solid
    topBar.style.color = contrastColor(themeColor1);
    topBar.style.borderBottom = `2px solid ${invertColor(themeColor1, 0.06)}`;
    embedContainer.style.background = themeColor2;      // solid
    embedContainer.style.color = contrastColor(themeColor2);
}

/* small helpers for contrast */
function hexToRgb(hex) {
    hex = hex.replace('#','');
    if (hex.length === 3) hex = hex.split('').map(c => c+c).join('');
    const num = parseInt(hex, 16);
    return [(num>>16)&255, (num>>8)&255, num&255];
}
function luminance(r,g,b) {
    const a = [r,g,b].map(v => {
        v /= 255;
        return v <= 0.03928 ? v/12.92 : Math.pow((v+0.055)/1.055, 2.4);
    });
    return 0.2126*a[0] + 0.7152*a[1] + 0.0722*a[2];
}
function contrastColor(hex) {
    const [r,g,b] = hexToRgb(hex);
    return luminance(r,g,b) > 0.5 ? '#000000' : '#ffffff';
}
function invertColor(hex, alpha=0.08) {
    // return a slightly lighter/darker border color by blending with black or white
    const [r,g,b] = hexToRgb(hex);
    return `rgba(${r},${g},${b},${alpha})`;
}

/* store theme */
function saveThemeToStorage() {
    localStorage.setItem('themeColor1', themeColor1);
    localStorage.setItem('themeColor2', themeColor2);
}

/* click sound (kept) */
function playClickSound() {
    const clickSound = new Audio("https://sevvy-wevvy.com/files/click1.mp3");
    clickSound.play().catch(()=>{/* ignore autoplay issues */});
}

/* settings UI */
function Settings() {
    playClickSound();
    embedContainer.className = 'settings';
    embedContainer.innerHTML = `
        <h2 style="font-family: Arial, sans-serif; color: ${contrastColor(themeColor2)};">Theme Settings</h2>
        <div class="color-picker">
            <div>
                <div id="pickr1"></div>
                <div style="opacity:0.7; font-family: Arial, sans-serif; color:${contrastColor(themeColor2)}">Top Bar Color</div>
            </div>
            <div>
                <div id="pickr2"></div>
                <div style="opacity:0.7; font-family: Arial, sans-serif; color:${contrastColor(themeColor2)}">Content Background</div>
            </div>
        </div>
        <label style="margin-top: 20px; color: ${contrastColor(themeColor2)}; font-family: Arial, sans-serif;">
            <input type="checkbox" id="catToggle" style="transform: scale(1.3); margin-right: 10px;">
            Enable Cat
        </label>
        <label style="margin-top: 20px; color: ${contrastColor(themeColor2)}; font-family: Arial, sans-serif;">
            Watched Usernames (comma-separated):
            <input type="text" id="watchedUsernames" style="margin-top: 5px; width: 100%; padding: 6px; font-family: Arial, sans-serif;" />
        </label>
        <label style="margin-top: 20px; color: ${contrastColor(themeColor2)}; font-family: Arial, sans-serif;">
            Music Volume:
            <input type="range" id="volumeSlider" min="0" max="1" step="0.01" style="width: 100%; margin-top: 5px;">
        </label>
        <button id="resetBtn">Reset to Default</button>
    `;

    document.getElementById('catToggle').checked = catEnabled;
    document.getElementById('catToggle').addEventListener('change', (e) => {
        catEnabled = e.target.checked;
        localStorage.setItem('catEnabled', catEnabled);
        cat.style.display = catEnabled ? 'block' : 'none';
        bubble.style.display = 'none';
    });

    document.getElementById('watchedUsernames').value = localStorage.getItem('watchedUsernames') || '';
    document.getElementById('watchedUsernames').addEventListener('input', (e) => {
        localStorage.setItem('watchedUsernames', e.target.value.trim());
    });

    const volumeSlider = document.getElementById('volumeSlider');
    const storedVolume = parseFloat(localStorage.getItem('musicVolume')) || 0.5;
    volumeSlider.value = storedVolume;
    if (audio) audio.volume = storedVolume;
    volumeSlider.addEventListener('input', (e) => {
        const vol = parseFloat(e.target.value);
        if (audio) audio.volume = vol;
        localStorage.setItem('musicVolume', vol);
    });

    document.getElementById('resetBtn').addEventListener('click', () => {
        playClickSound();
        themeColor1 = defaultColor1;
        themeColor2 = defaultColor2;
        catEnabled = true;
        pickr1.setColor(themeColor1);
        pickr2.setColor(themeColor2);
        updateTheme();
        saveThemeToStorage();
        localStorage.setItem('catEnabled', 'true');
        document.getElementById('catToggle').checked = true;
        cat.style.display = 'block';
        bubble.style.display = 'none';
    });

    /* Pickr color pickers — still usable but set to apply solid colors (not gradients) */
    const pickr1 = Pickr.create({
        el: '#pickr1',
        theme: 'classic',
        default: themeColor1,
        components: { preview: true, opacity: false, hue: true, interaction: { hex: true, input: true, save: true } }
    });

    const pickr2 = Pickr.create({
        el: '#pickr2',
        theme: 'classic',
        default: themeColor2,
        components: { preview: true, opacity: false, hue: true, interaction: { hex: true, input: true, save: true } }
    });

    pickr1.on('save', (color) => {
        themeColor1 = color.toHEXA().toString();
        updateTheme();
        saveThemeToStorage();
    });

    pickr2.on('save', (color) => {
        themeColor2 = color.toHEXA().toString();
        updateTheme();
        saveThemeToStorage();
    });
}

/* inventory loader (unchanged) */
function loadInventory() {
    embedContainer.classList.remove('settings');
    embedContainer.removeAttribute('style');
    embedContainer.style.background = themeColor2 || '#fff';
    embedContainer.style.color = contrastColor(themeColor2 || '#fff');
    embedContainer.innerHTML = userId
        ? `<iframe src="https://app.blobtown.com/inventory/?blobt=${userId}" allowfullscreen></iframe>`
        : `<div style="color: red; font-size: 20px; text-align: center; margin-top: 30px;">
            ⚠ No ID provided in the URL. Please add <code>?id=YOURID</code> to the URL.
           </div>`;
}

/* removed goBrowser / goChat usage from UI (buttons removed). Keeping functions in case you want them later. */
function goBrowser() { playClickSound(); embedContainer.classList.remove('settings'); embedContainer.style.background = themeColor2 || '#fff'; embedContainer.innerHTML = `<iframe src="https://sevvy-wevvy.com/mods/blobtown/sev/browser/" allowfullscreen></iframe>`; }
function goChat() { playClickSound(); embedContainer.classList.remove('settings'); embedContainer.style.background = themeColor2 || '#fff'; embedContainer.innerHTML = `<iframe src="https://gggravity.org/chatroom/use.php" allowfullscreen></iframe>`; clearChatNotification(); }

/* keep other functions (kill/refresh) if you need them later, but UI does not expose them */
function kill() { window.location.href = `https://sevvy-wevvy.com/mods/blobtown?id=${userId}`; }
function refresh() { location.reload(); }

/* clock */
function updateClock() {
    const now = new Date();
    const h = now.getHours().toString().padStart(2, '0');
    const m = now.getMinutes().toString().padStart(2, '0');
    const s = now.getSeconds().toString().padStart(2, '0');
    document.getElementById('clock').textContent = `${h}:${m}:${s}`;
}

/* cat, notifications, etc. — unchanged */
let catX = 200, catY = 200, velX = 0, velY = 0;
let climbing = false, direction = 1, grounded = false;
const gravity = 0.6, friction = 0.8, jumpPower = -10, moveSpeed = 2;
function showBubble(text) {
    if (!catEnabled) return;
    bubble.textContent = text;
    bubble.style.display = 'block';
    setTimeout(() => { if (catEnabled) bubble.style.display = 'none'; }, 3000);
}
function updateCat() {
    if (!catEnabled) return requestAnimationFrame(updateCat);
    if (Math.random() < 0.01) { direction *= -1; cat.style.transform = `scaleX(${direction})`; }
    if (grounded && Math.random() < 0.01) { velY = jumpPower; grounded = false; }
    if (Math.random() < 0.005) {
        const phrases = ["Meow!", "Let me up!", "Zoomies!", "Wheee~", "Mrowww~", "Wall climb!"];
        showBubble(phrases[Math.floor(Math.random() * phrases.length)]);
    }
    if ((catX <= 0 || catX >= window.innerWidth - 100) && Math.random() < 0.2) { climbing = true; velY = -2; }
    if (climbing && catY <= 100) { climbing = false; velY = 1; }
    if (!climbing) velX = direction * moveSpeed;
    if (!climbing) velY += gravity;
    else velY = -1;
    catX += velX;
    catY += velY;
    velX *= friction;
    const groundY = window.innerHeight - 100;
    if (catX < 0) { catX = 0; direction = 1; }
    if (catX > window.innerWidth - 100) { catX = window.innerWidth - 100; direction = -1; }
    if (catY >= groundY) { catY = groundY; velY = 0; grounded = true; climbing = false; }
    if (catY < 0) { catY = 0; velY = 1; }
    cat.style.left = `${catX}px`;
    cat.style.top = `${catY}px`;
    bubble.style.left = `${catX}px`;
    bubble.style.top = `${catY - 30}px`;
    requestAnimationFrame(updateCat);
}

/* watched user check and notifications — kept intact */
let lastTextLogLine = "";
async function checkWatchedUsernames() {
    const code = localStorage.code || "1234";
    const rawInput = localStorage.getItem('watchedUsernames') || '';
    const watched = rawInput.split(',').map(name => name.trim().toLowerCase()).filter(Boolean);
    if (watched.length === 0) return;
    try {
        const chatRes = await fetch(`https://gggravity.org/chatroom/chatlogs/chatlog_${code}.txt?` + Date.now());
        const userRes = await fetch(`https://gggravity.org/chatroom/userlogs/userlog_${code}.txt?` + Date.now());
        if (!chatRes.ok || !userRes.ok) return;
        const chatData = await chatRes.text();
        const userData = await userRes.text();
        const chatLines = chatData.split('\n').map(l => l.trim()).filter(Boolean);
        const userLines = userData.split('\n').map(l => l.trim().toLowerCase()).filter(Boolean);
        const latestChat = chatLines[chatLines.length - 1];
        const latestUser = userLines[userLines.length - 1];
        if (!lastTextLogLine) { lastTextLogLine = latestChat; return; }
        if (latestChat !== lastTextLogLine) {
            lastTextLogLine = latestChat;
            for (const name of watched) {
                if (latestUser === name) {
                    showNotification(`Your friend "${name}" sent a message!`);
                    showChatNotification();
                    break;
                }
            }
        }
    } catch (e) { console.warn("User check failed:", e); }
}

function showNotification(text) {
    const clickSound = new Audio("https://sevvy-wevvy.com/files/notifi1.mp3");
    clickSound.play().catch(()=>{});
    notifMessage.textContent = text;
    notifMessage.style.display = 'block';
    clearTimeout(showNotification.timeout);
    showNotification.timeout = setTimeout(() => { notifMessage.style.display = 'none'; }, 4000);
}
function showChatNotification() {
    // no chat button in UI, but we keep this so user notifications still work logically
}
function clearChatNotification() {
    // kept
}

setInterval(checkWatchedUsernames, 1000);
cat.style.display = catEnabled ? 'block' : 'none';
updateCat();
setInterval(updateClock, 1000);
updateClock();
updateTheme();
loadInventory();

/* audio sync code unchanged; kept at end of file */
</script>
</body>
<script>
let audio;
let syncInterval;
(() => {
    let currentVersion = '';

    async function fetchSync() {
        try {
            const res = await fetch('https://gggravity.org/synced-music/song.php', {
                mode: 'cors',
                cache: 'no-cache',
            });
            if (!res.ok) throw new Error(`Sync fetch failed with status ${res.status}`);
            return await res.json();
        } catch (e) {
            console.error('Error fetching sync:', e);
            return null;
        }
    }

    async function startAudioPlayback() {
        let currentAudioSrc = '';

        async function checkAndSync() {
            const syncData = await fetchSync();
            if (!syncData || !syncData.version) {
                console.warn('Invalid sync data or missing version. Retrying next interval.');
                return;
            }

            if (syncData.version === currentVersion) return;
            currentVersion = syncData.version;

            const newAudioSrc = syncData.url + '?t=' + syncData.start_time;
            const now = Date.now() / 1000;
            const expectedOffset = (now - syncData.start_time) % syncData.length;

            console.log(`[Sync] Version changed. New song: ${syncData.song}`);
            currentAudioSrc = newAudioSrc;

            audio.src = newAudioSrc;
            audio.load();

            const playNewSong = async () => {
                const playTimeOffset = (Date.now() / 1000 - syncData.start_time) % syncData.length;
                audio.currentTime = playTimeOffset;

                try {
                    await audio.play();
                    console.log(`[Audio] Playing ${syncData.song} at offset ${playTimeOffset.toFixed(2)}s`);
                } catch (e) {
                    console.error(`[Audio] Failed to play new song. Browser may require user interaction.`, e);
                }
                audio.removeEventListener('canplay', playNewSong);
            };

            audio.addEventListener('canplay', playNewSong);
        }

        await checkAndSync();

        if (syncInterval) clearInterval(syncInterval);
        syncInterval = setInterval(checkAndSync, 2000);
    }

    function initializeAudioOnInteraction() {
        console.log('[Init] User interaction detected. Initializing audio element.');

        audio = new Audio();
        audio.volume = parseFloat(localStorage.getItem('musicVolume')) || 0.5;
        audio.crossOrigin = 'anonymous';
        audio.preload = 'auto';

        audio.addEventListener('error', (e) => {
            console.error('[Audio Error] An error occurred with the audio element:', e);
        });

        startAudioPlayback();
    }

    const events = ['click', 'keydown', 'touchstart'];
    events.forEach(e => document.addEventListener(e, initializeAudioOnInteraction, { once: true }));

})();
</script>
<iframe id="profileIframe" src="https://gggravity.org/gravity-accounts/profile" allowtransparency="true"></iframe>

<button id="toggleBtn" title="Toggle Profile">
  <img src="https://gggravity.org/gravity-accounts/overlay/user.png" alt="User" />
</button>

<style>
  #profileIframe {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0.95);
  width: 90vw;
  height: 80vh;
  max-width: 1200px;
  max-height: 90vh;
  border: none;
  border-radius: 20px;
  background: white;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.35);
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.4s ease, transform 0.4s ease;
  z-index: 999;
}

#profileIframe.active {
  opacity: 1;
  pointer-events: auto;
  transform: translate(-50%, -50%) scale(1);
}
</style>

<script>
  const btn = document.getElementById("toggleBtn");
  const iframe = document.getElementById("profileIframe");

  btn.onclick = () => {
    iframe.classList.toggle("active");
  };
</script>

</html>
