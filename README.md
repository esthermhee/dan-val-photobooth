# dan-val-photobooth
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Dan & Val’s Photobooth</title>

<style>
  body {
    font-family: Georgia, serif;
    background: #f6f2ec;
    text-align: center;
    padding: 20px;
  }

  h1 { margin-bottom: 4px; }

  #booth {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 14px;
  }

  /* SIDE PROGRESS FRAMES */
  #progress {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .step {
    width: 16px;
    height: 80px;
    border: 1px solid #000;
    background: #fff;
  }

  .step.filled {
    background: #000;
  }

  #container {
    position: relative;
  }

  video {
    width: 300px;
    height: 375px; /* 4:5 preview */
    object-fit: cover;
    transform: scaleX(-1);
    filter: grayscale(100%);
    border: 2px solid #000;
    background: #000;
  }

  #countdown {
    position: absolute;
    inset: 0;
    display: none;
    align-items: center;
    justify-content: center;
    font-size: 72px;
    font-weight: bold;
    color: white;
    background: rgba(0,0,0,0.35);
  }

  button, a {
    margin-top: 14px;
    padding: 12px 20px;
    font-size: 16px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    text-decoration: none;
  }

  button {
    background: #000;
    color: white;
  }

  a {
    background: #000;
    color: white;
    display: none;
  }

  canvas { display: none; }
</style>
</head>

<body>

<h1>Dan & Val’s Photobooth</h1>
<p>4 shots · black & white · instant keepsake</p>

<div id="booth">
  <div id="progress">
    <div class="step"></div>
    <div class="step"></div>
    <div class="step"></div>
    <div class="step"></div>
  </div>

  <div id="container">
    <video id="video" autoplay playsinline></video>
    <div id="countdown"></div>
  </div>
</div>

<br>
<button id="start">Start</button><br>
<a id="download">Download Photostrip</a>

<canvas id="photoCanvas"></canvas>
<canvas id="stripCanvas"></canvas>

<script>
const video = document.getElementById("video");
const photoCanvas = document.getElementById("photoCanvas");
const stripCanvas = document.getElementById("stripCanvas");
const startBtn = document.getElementById("start");
const countdown = document.getElementById("countdown");
const download = document.getElementById("download");
const steps = document.querySelectorAll(".step");

const photoCtx = photoCanvas.getContext("2d");
const stripCtx = stripCanvas.getContext("2d");

const PHOTOS = [];
const COUNT = 4;
const W = 480;
const H = 600;

navigator.mediaDevices.getUserMedia({ video: { aspectRatio: 4/5 } })
  .then(stream => video.srcObject = stream)
  .catch(() => alert("Camera access required"));

async function wait(ms) {
  return new Promise(r => setTimeout(r, ms));
}

async function countdownShot() {
  for (let i = 3; i > 0; i--) {
    countdown.textContent = i;
    countdown.style.display = "flex";
    await wait(1000);
  }
  countdown.style.display = "none";
  takePhoto();
}

function takePhoto() {
  photoCanvas.width = W;
  photoCanvas.height = H;

  photoCtx.save();
  photoCtx.translate(W, 0);
  photoCtx.scale(-1, 1);
  photoCtx.filter = "grayscale(100%)";
  photoCtx.drawImage(video, 0, 0, W, H);
  photoCtx.restore();

  PHOTOS.push(photoCanvas.toDataURL("image/png"));
  steps[PHOTOS.length - 1].classList.add("filled");
}

function buildStrip() {
  const gap = 12;
  stripCanvas.width = W + 40;
  stripCanvas.height = COUNT * H + gap * (COUNT - 1) + 40;

  stripCtx.fillStyle = "#fff";
  stripCtx.fillRect(0, 0, stripCanvas.width, stripCanvas.height);

  PHOTOS.forEach((src, i) => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      const x = 20;
      const y = 20 + i * (H + gap);

      stripCtx.drawImage(img, x, y, W, H);
      stripCtx.strokeStyle = "#000";
      stripCtx.lineWidth = 1;
      stripCtx.strokeRect(x, y, W, H);

      if (i === COUNT - 1) {
        const final = stripCanvas.toDataURL("image/png");
        download.href = final;
        download.download = "Dan-Val-Photobooth.png";
        download.style.display = "inline-block";
      }
    };
  });
}

startBtn.onclick = async () => {
  PHOTOS.length = 0;
  download.style.display = "none";
  steps.forEach(s => s.classList.remove("filled"));

  for (let i = 0; i < COUNT; i++) {
    await countdownShot();
    await wait(700);
  }

  buildStrip();
};
</script>

</body>
</html>
