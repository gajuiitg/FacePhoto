<html lang="en">
<head>
<meta name="google-site-verification" content="LvwRw__hvxynJhfxTBfeEEcX60bRv7MIyI2Un6NXQsQ">
<meta charset="UTF-8">
<title>Drive Person Finder</title>
<script src="https://accounts.google.com/gsi/client" async defer></script>
<script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js" onerror="window.faceApiLoadError = true"></script>
<style>
  :root {
    --bg: #0f1115;
    --card: #171a21;
    --accent: #4f8cff;
    --good: #3ddc84;
    --text: #e8eaed;
    --muted: #9aa0a6;
    --border: #2a2e37;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    background: var(--bg);
    color: var(--text);
    padding: 24px 16px 60px;
  }
  .wrap { max-width: 760px; margin: 0 auto; }
  h1 { font-size: 20px; margin-bottom: 4px; }
  p.sub { color: var(--muted); font-size: 14px; margin-top: 0; }

  .panel {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 18px;
    margin-bottom: 16px;
  }
  label { font-size: 13px; color: var(--muted); display: block; margin-bottom: 6px; }
  input[type=text] {
    width: 100%;
    padding: 10px 12px;
    border-radius: 8px;
    border: 1px solid var(--border);
    background: #10131a;
    color: var(--text);
    font-size: 14px;
    margin-bottom: 12px;
  }
  input[type=range] { width: 100%; }
  button {
    background: var(--accent);
    color: white;
    border: none;
    padding: 10px 16px;
    border-radius: 8px;
    font-size: 14px;
    cursor: pointer;
    font-weight: 600;
  }
  button:disabled { opacity: 0.5; cursor: not-allowed; }
  button.secondary {
    background: transparent;
    border: 1px solid var(--border);
    color: var(--text);
  }
  .row { display: flex; gap: 10px; flex-wrap: wrap; align-items: center; }

  .status { font-size: 13px; color: var(--muted); margin-top: 8px; white-space: pre-line; }
  .error { color: #ff6b6b; }

  #refPreview {
    width: 90px; height: 90px; object-fit: cover;
    border-radius: 8px; border: 1px solid var(--border);
    display: none; margin-top: 10px;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
    gap: 12px;
  }
  .photo-card {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 10px;
    overflow: hidden;
    text-decoration: none;
    color: var(--text);
    display: block;
    position: relative;
  }
  .photo-card img {
    width: 100%;
    height: 110px;
    object-fit: cover;
    display: block;
    background: #222;
  }
  .photo-card .meta { padding: 8px 10px; }
  .photo-card .name {
    font-size: 12px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .photo-card .match {
    font-size: 11px;
    color: var(--good);
  }
  .badge {
    position: absolute; top: 6px; right: 6px;
    background: rgba(0,0,0,0.65);
    color: var(--good);
    font-size: 11px;
    padding: 2px 6px;
    border-radius: 6px;
  }

  details summary {
    cursor: pointer;
    color: var(--muted);
    font-size: 13px;
    margin-bottom: 8px;
  }
  code {
    background: #10131a;
    padding: 1px 5px;
    border-radius: 4px;
    font-size: 12px;
  }
  .progress-bar {
    height: 6px;
    background: #10131a;
    border-radius: 4px;
    overflow: hidden;
    margin-top: 8px;
  }
  .progress-fill {
    height: 100%;
    background: var(--accent);
    width: 0%;
    transition: width 0.2s;
  }
</style>
</head>
<body>
<div class="wrap">
  <h1>🧑‍🤝‍🧑 Drive Person Finder</h1>
  <p class="sub">Upload one clear photo of a person, and this finds matching photos of them inside a Drive folder. Everything runs in your browser — no photo is uploaded to any server.</p>

  <div class="panel">
    <details>
      <summary>One-time setup (Google API credentials)</summary>
      <p style="font-size:13px; color:var(--muted); line-height:1.5;">
        1. Go to <code>console.cloud.google.com</code> → create/select a project.<br>
        2. Enable the <b>Google Drive API</b>.<br>
          3. Create credentials → <b>OAuth client ID</b> → type <b>Web application</b> →
            under "Authorized JavaScript origins", add the exact origin where this page runs:
            <code>https://gajuiitg.github.io</code> for the deployed site, or
            <code>http://localhost:8000</code> for local testing. Do not add the page path.<br>
        4. Create an <b>API key</b> too.<br>
        5. Paste both below. They stay in your browser only.
      </p>
    </details>

    <label for="clientId">OAuth Client ID</label>
    <input type="text" id="clientId" placeholder="xxxxxxxx.apps.googleusercontent.com" autocomplete="off">

    <label for="apiKey">API Key</label>
    <input type="text" id="apiKey" placeholder="AIzaSy..." autocomplete="off">

    <label for="folderId">Folder ID</label>
    <input type="text" id="folderId" value="1N64KZLDRgeIVfUHtZ4A9diF4fRjwwp4Z">

    <label for="refInput">Reference photo of the person</label>
    <input type="file" id="refInput" accept="image/*">
    <img id="refPreview" alt="reference preview">

    <label for="threshold" style="margin-top:12px;">Match strictness (lower = stricter match)</label>
    <input type="range" id="threshold" min="0.3" max="0.7" step="0.02" value="0.5">
    <div class="status" id="thresholdLabel">Threshold: 0.50</div>

    <div class="row" style="margin-top:14px;">
      <button id="signInBtn">Sign in &amp; Search</button>
      <button id="signOutBtn" class="secondary" style="display:none;">Sign out</button>
    </div>
    <div class="progress-bar"><div class="progress-fill" id="progressFill"></div></div>
    <div class="status" id="status">Loading face recognition models…</div>
  </div>

  <div class="grid" id="results"></div>
</div>

<script>
const MODEL_URL = 'https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js@master/weights';
let modelsReady = false;
let tokenClient;
let accessToken = null;
let refDescriptor = null;

function setStatus(msg, isError) {
  const el = document.getElementById('status');
  el.textContent = msg;
  el.className = 'status' + (isError ? ' error' : '');
}
function setProgress(pct) {
  document.getElementById('progressFill').style.width = pct + '%';
}
function extractFolderId(raw) {
  const match = raw.match(/[-\w]{20,}/);
  return match ? match[0] : raw.trim();
}

function normalizeCredential(value) {
  return value.replace(/\s+/g, '').trim();
}

async function loadModels() {
  try {
    if (typeof faceapi === 'undefined') {
      throw new Error('face-api.js library failed to load from cdn.jsdelivr.net');
    }
    // tinyFaceDetector is a single small file and loads far more reliably
    // from CDNs than ssdMobilenetv1 (which needs several shard files).
    await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
    await faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL);
    await faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL);
    modelsReady = true;
    setStatus('Ready. Upload a reference photo and sign in to search.');
  } catch (e) {
    console.error('Model load failed:', e);
    setStatus('Failed to load face models: ' + e.message + ' (check console for details, and see if cdn.jsdelivr.net is reachable)', true);
  }
}
if (typeof faceapi === 'undefined') {
  setStatus('Failed to load face models: face-api.js library is unavailable. Check whether cdn.jsdelivr.net is reachable.', true);
} else {
  loadModels();
}

document.getElementById('threshold').addEventListener('input', (e) => {
  document.getElementById('thresholdLabel').textContent = 'Threshold: ' + e.target.value;
});

const detectorOptions = typeof faceapi !== 'undefined'
  ? new faceapi.TinyFaceDetectorOptions({ inputSize: 416, scoreThreshold: 0.3 })
  : null;

document.getElementById('refInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;
  const preview = document.getElementById('refPreview');
  preview.src = URL.createObjectURL(file);
  preview.style.display = 'block';

  if (!modelsReady) {
    setStatus('Still loading face models, try again in a moment…', true);
    return;
  }

  refDescriptor = null;
  setStatus('Analyzing reference photo…');

  try {
    const img = await faceapi.bufferToImage(file);
    const detection = await faceapi
      .detectSingleFace(img, detectorOptions)
      .withFaceLandmarks()
      .withFaceDescriptor();

    if (!detection) {
      setStatus('No face detected in that photo. Try a clearer, front-facing, well-lit photo (and avoid HEIC files — convert to JPG/PNG first).', true);
      return;
    }
    refDescriptor = detection.descriptor;
    setStatus('Reference face captured. Ready to search.');
  } catch (err) {
    console.error('Reference photo analysis failed:', err);
    setStatus('Error analyzing photo: ' + err.message + ' (if this is a .heic file, convert it to .jpg first)', true);
  }
});

function initTokenClient() {
  const clientIdInput = document.getElementById('clientId');
  const clientId = normalizeCredential(clientIdInput.value);
  clientIdInput.value = clientId;
  if (!clientId) {
    setStatus('Enter your OAuth Client ID first.', true);
    return null;
  }
  if (!/^[0-9]+-[a-zA-Z0-9_-]+\.apps\.googleusercontent\.com$/.test(clientId)) {
    setStatus('OAuth Client ID is invalid. Paste the complete value ending in .apps.googleusercontent.com.', true);
    return null;
  }
  try {
    return google.accounts.oauth2.initTokenClient({
      client_id: clientId,
      scope: 'https://www.googleapis.com/auth/drive.readonly',
      callback: (resp) => {
        if (resp.error) {
          setStatus('Sign-in failed: ' + resp.error + '. Check that this exact client ID exists in Google Cloud and that this site origin is authorized.', true);
          return;
        }
        accessToken = resp.access_token;
        document.getElementById('signOutBtn').style.display = 'inline-block';
        runSearch();
      },
    });
  } catch (err) {
    setStatus('Could not initialize Google sign-in: ' + err.message + '. Use a Web application OAuth client from Google Cloud, not an API key or another client type.', true);
    return null;
  }
}

async function listImagesInFolder(folderId, apiKey) {
  const query = encodeURIComponent(
    `'${folderId}' in parents and mimeType contains 'image/' and trashed = false`
  );
  const fields = encodeURIComponent('files(id,name,mimeType,webViewLink,thumbnailLink)');
  const url = `https://www.googleapis.com/drive/v3/files?q=${query}&fields=${fields}&pageSize=200&key=${apiKey}`;
  const res = await fetch(url, { headers: { Authorization: `Bearer ${accessToken}` } });
  const data = await res.json();
  if (data.error) throw new Error(data.error.message);
  return data.files || [];
}

async function fetchImageBlob(fileId) {
  const url = `https://www.googleapis.com/drive/v3/files/${fileId}?alt=media`;
  const res = await fetch(url, { headers: { Authorization: `Bearer ${accessToken}` } });
  if (!res.ok) throw new Error('Could not download file ' + fileId);
  return await res.blob();
}

async function runSearch() {
  const resultsEl = document.getElementById('results');
  resultsEl.innerHTML = '';
  setProgress(0);

  if (!modelsReady) {
    setStatus('Face models still loading, please wait…', true);
    return;
  }
  if (!refDescriptor) {
    setStatus('Upload a reference photo with a visible face first.', true);
    return;
  }

  const apiKeyInput = document.getElementById('apiKey');
  const apiKey = normalizeCredential(apiKeyInput.value);
  apiKeyInput.value = apiKey;
  const folderId = extractFolderId(document.getElementById('folderId').value);
  const threshold = parseFloat(document.getElementById('threshold').value);

  if (!apiKey) { setStatus('Enter your API Key first.', true); return; }
  if (!folderId) { setStatus('Enter a folder ID or link.', true); return; }

  if (window.location.protocol === 'file:') {
    setStatus('Run this page from http://localhost, not directly as a file. Example: py -m http.server 8000 --directory "C:\\Users\\THIS PC\\Downloads"', true);
    return;
  }

  setStatus('Listing photos in folder…');
  let files;
  try {
    files = await listImagesInFolder(folderId, apiKey);
  } catch (err) {
    setStatus('Error listing files: ' + err.message, true);
    return;
  }

  if (files.length === 0) {
    setStatus('No photos found in that folder.');
    return;
  }

  let matches = 0;
  for (let i = 0; i < files.length; i++) {
    const f = files[i];
    setStatus(`Scanning ${i + 1} of ${files.length}: ${f.name}`);
    setProgress(Math.round(((i + 1) / files.length) * 100));

    try {
      const blob = await fetchImageBlob(f.id);
      const img = await faceapi.bufferToImage(blob);
      const detections = await faceapi
        .detectAllFaces(img, detectorOptions)
        .withFaceLandmarks()
        .withFaceDescriptors();

      let bestDist = Infinity;
      for (const d of detections) {
        const dist = faceapi.euclideanDistance(refDescriptor, d.descriptor);
        if (dist < bestDist) bestDist = dist;
      }

      if (bestDist <= threshold) {
        matches++;
        addResultCard(f, bestDist);
      }
    } catch (err) {
      console.warn('Skipping', f.name, err.message);
    }
  }

  setStatus(`Done. ${matches} matching photo(s) out of ${files.length} scanned.`);
}

function addResultCard(file, distance) {
  const resultsEl = document.getElementById('results');
  const card = document.createElement('a');
  card.href = file.webViewLink;
  card.target = '_blank';
  card.rel = 'noopener noreferrer';
  card.className = 'photo-card';

  const confidence = Math.max(0, Math.round((1 - distance) * 100));

  const img = document.createElement('img');
  img.src = file.thumbnailLink || '';
  img.alt = file.name;
  img.onerror = () => { img.style.display = 'none'; };

  const badge = document.createElement('div');
  badge.className = 'badge';
  badge.textContent = confidence + '%';

  const meta = document.createElement('div');
  meta.className = 'meta';
  meta.innerHTML = `<div class="name">${file.name}</div><div class="match">match confidence</div>`;

  card.appendChild(img);
  card.appendChild(badge);
  card.appendChild(meta);
  resultsEl.appendChild(card);
}

document.getElementById('signInBtn').addEventListener('click', () => {
  if (window.location.protocol === 'file:') {
    setStatus('Google sign-in requires a web origin. Start a local server, then open http://localhost:8000/facecheck.html. Add that exact origin to your OAuth client settings.', true);
    return;
  }
  if (!window.google?.accounts?.oauth2) {
    setStatus('Google sign-in is still loading. Check that accounts.google.com is reachable, then try again.', true);
    return;
  }
  if (!refDescriptor) {
    setStatus('Upload a reference photo with a visible face before searching.', true);
    return;
  }
  tokenClient = initTokenClient();
  if (!tokenClient) return;

  if (accessToken) {
    runSearch();
  } else {
    tokenClient.requestAccessToken({ prompt: 'consent' });
  }
});

document.getElementById('signOutBtn').addEventListener('click', () => {
  if (accessToken) {
    google.accounts.oauth2.revoke(accessToken, () => {
      accessToken = null;
      document.getElementById('signOutBtn').style.display = 'none';
      document.getElementById('results').innerHTML = '';
      setStatus('Signed out.');
    });
  }
});
</script>
</body>
</html>
