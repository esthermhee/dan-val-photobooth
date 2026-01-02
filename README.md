<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
<title>Dan & Val’s Photobooth</title>

<link href="https://fonts.googleapis.com/css2?family=Homemade+Apple&display=swap" rel="stylesheet">

<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    overflow: hidden;
    background: #f6f2ec;
    font-family: Georgia, serif;
    touch-action: manipulation;
  }

  body {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  h1 { margin-bottom: 6px; }
  p { margin-top: 0; }

  #booth {
    display: flex;
    align-items: center;
    gap: 16px;
  }

  /* SIDE PROGRESS */
  #progress {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .step {
    width: 18px;
    height: 90px;
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
    width: 320px;
    height: 400px;
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
    font-size: 80px;
    font-weight: bold;
    color: white;
    background: rgba(0,0,0,0.35);
  }

  /* FLASH */
  #flash {
    position: fixed;
    inset: 0;
    background: white;
    opacity: 0;
    pointer-events: none;
    z-index: 999;
  }

  button, a {
    margin-top: 16px;
    padding: 16px 28px;
    font-size: 18px;
    border-radius: 10px;
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
<p>Tap · Pose · Repeat</p>

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

<button id="start">Start</button>
<a id="download">Download Photostrip</a>

<div id="flash"></div>

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
const flash = document.getElementById("flash");

const photoCtx = photoCanvas.getContext("2d");
const stripCtx = stripCanvas.getContext("2d");

const PHOTOS = [];
const COUNT = 4;
const W = 480;
const H = 600;

navigator.mediaDevices.getUserMedia({
  video: { aspectRatio: 4/5 }
}).then(stream => video.srcObject = stream);

const wait = ms => new Promise(r => setTimeout(r, ms));

function addGrain(ctx, w, h) {
  const imageData = ctx.getImageData(0, 0, w, h);
  for (let i = 0; i < imageData.data.length; i += 4) {
    const noise = (Math.random() - 0.5) * 18;
    imageData.data[i] += noise;
    imageData.data[i+1] += noise;
    imageData.data[i+2] += noise;
  }
  ctx.putImageData(imageData, 0, 0);
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

function flashEffect() {
  flash.style.opacity = 1;
  setTimeout(() => flash.style.opacity = 0, 120);
}

function takePhoto() {
  flashEffect();

  photoCanvas.width = W;
  photoCanvas.height = H;

  photoCtx.save();
  photoCtx.translate(W, 0);
  photoCtx.scale(-1, 1);
  photoCtx.filter = "grayscale(100%)";
  photoCtx.drawImage(video, 0, 0, W, H);
  photoCtx.restore();

  addGrain(photoCtx, W, H);

  PHOTOS.push(photoCanvas.toDataURL("image/png"));
  steps[PHOTOS.length - 1].classList.add("filled");
}

function buildStrip() {
  const gap = 12;
  const padding = 24;

  stripCanvas.width = W + padding * 2;
  stripCanvas.height = COUNT * H + gap * (COUNT - 1) + 140;

  stripCtx.fillStyle = "#000";
  stripCtx.fillRect(0, 0, stripCanvas.width, stripCanvas.height);

  PHOTOS.forEach((src, i) => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      const x = padding;
      const y = padding + i * (H + gap);

      stripCtx.drawImage(img, x, y, W, H);
      stripCtx.strokeStyle = "#fff";
      stripCtx.lineWidth = 1;
      stripCtx.strokeRect(x, y, W, H);

      if (i === COUNT - 1) {
        stripCtx.fillStyle = "#fff";
        stripCtx.textAlign = "center";
        stripCtx.font = "36px 'Homemade Apple'";
        stripCtx.fillText("Dan & Val", stripCanvas.width / 2, stripCanvas.height - 70);
        stripCtx.font = "24px 'Homemade Apple'";
        stripCtx.fillText("24.01.26", stripCanvas.width / 2, stripCanvas.height - 36);

        const final = stripCanvas.toDataURL("image/png");
        download.href = final;
        download.download = "Dan-Val-Photostrip.png";
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

