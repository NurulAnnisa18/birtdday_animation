# birtdday_animation
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Happy Birthday Fireworks Animation</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap');

  :root {
    --primary-color: #ff416c;
    --secondary-color: #ff4b2b;
    --balloon-colors: #ff4b2b, #ff416c, #ff6f91, #fcbad3, #ff9a76;
    --flower-colors: #ffb6c1, #ffc0cb, #ff69b4, #ff1493, #db7093;
  }

  body {
    margin: 0;
    height: 100vh;
    background: radial-gradient(ellipse at center, #000000, #0a0a0a);
    font-family: 'Playfair Display', serif;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    user-select: none;
    position: relative;
    color: white;
  }

  canvas#fireworks-canvas {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    pointer-events: none;
    z-index: 20;
  }

  /* Text container and styles */
  h1 {
    font-size: 4rem;
    font-weight: 700;
    margin-bottom: 1rem;
    overflow: visible;
    display: flex;
    gap: 0.12em;
    position: relative;
    user-select: none;
    z-index: 25;
  }

  h1 span {
    opacity: 0;
    transform: translateY(20px);
    display: inline-block;
    animation-fill-mode: forwards;
  }

  /* Balloon styles */
  .balloon {
    position: absolute;
    bottom: -150px;
    width: 60px;
    height: 80px;
    background: var(--balloon-colors);
    border-radius: 60% 60% 60% 60% / 70% 70% 30% 30%;
    opacity: 0.85;
    filter: drop-shadow(0 3px 3px rgba(0,0,0,0.8));
    animation-timing-function: ease-in-out;
    animation-name: floatUp;
    animation-iteration-count: infinite;
    z-index: 10;
  }

  /* Balloon string */
  .balloon::after {
    content: '';
    position: absolute;
    bottom: -20px;
    left: 50%;
    width: 2px;
    height: 20px;
    background-color: #333;
    transform: translateX(-50%);
    filter: drop-shadow(0 1px 1px rgba(0,0,0,0.5));
  }

  @keyframes floatUp {
    0% {
      transform: translateY(0) rotate(0deg);
      opacity: 0.85;
    }
    50% {
      transform: translateY(-40px) rotate(10deg);
      opacity: 1;
    }
    100% {
      transform: translateY(-80vh) rotate(-10deg);
      opacity: 0;
    }
  }

  /* Flower styles */
  .flower {
    position: absolute;
    bottom: -80px;
    width: 50px;
    height: 50px;
    filter: drop-shadow(0 2px 2px rgba(0,0,0,0.8));
    animation-name: floatFlower;
    animation-timing-function: ease-in-out;
    animation-iteration-count: infinite;
    cursor: default;
    z-index: 10;
  }

  /* Flower petals */
  .petal {
    position: absolute;
    width: 20px;
    height: 30px;
    background-color: var(--flower-petal-color, #ff69b4);
    border-radius: 50% / 70%;
    transform-origin: center bottom;
  }

  /* Four petals rotated */
  .petal1 { top: 5px; left: 15px; transform: rotate(0deg); }
  .petal2 { top: 15px; left: 28px; transform: rotate(45deg); }
  .petal3 { top: 20px; left: 10px; transform: rotate(-45deg); }
  .petal4 { top: 10px; left: 0;  transform: rotate(-90deg); }

  /* Flower center */
  .center {
    position: absolute;
    top: 18px;
    left: 18px;
    width: 15px;
    height: 15px;
    background-color: #fff1a8;
    border-radius: 50%;
    box-shadow: 0 0 8px #fffbbd;
  }

  @keyframes floatFlower {
    0% {
      transform: translateY(0) translateX(0) rotate(0deg);
      opacity: 0.9;
    }
    50% {
      transform: translateY(-35px) translateX(6px) rotate(7deg);
      opacity: 1;
    }
    100% {
      transform: translateY(-70vh) translateX(0) rotate(-7deg);
      opacity: 0;
    }
  }
</style>
</head>
<body>
  <h1 id="birthday-text" aria-label="Happy Birthday pikachuuuu!"></h1>

  <canvas id="fireworks-canvas"></canvas>

  <script>
    // Fireworks effect using canvas
    const canvas = document.getElementById('fireworks-canvas');
    const ctx = canvas.getContext('2d');
    let cw, ch;
    let fireworks = [];
    let particles = [];
    let hue = 120;
    let limiterTotal = 1; // Reduced to fire more often
    let limiterTick = 0;
    let timerTotal = 10; // Reduced for faster fireworks launching
    let timerTick = 0;
    let simultaneousFireworks = 3; // Number of fireworks launched each interval

    function setCanvasSize() {
      cw = window.innerWidth;
      ch = window.innerHeight;
      canvas.width = cw;
      canvas.height = ch;
    }
    setCanvasSize();
    window.addEventListener('resize', setCanvasSize);

    class Firework {
      constructor(sx, sy, tx, ty) {
        this.x = sx;
        this.y = sy;
        this.sx = sx;
        this.sy = sy;
        this.tx = tx;
        this.ty = ty;
        this.distanceToTarget = getDistance(sx, sy, tx, ty);
        this.distanceTraveled = 0;
        this.coordinates = [];
        this.coordinateCount = 3;
        while(this.coordinateCount--) {
          this.coordinates.push([this.x, this.y]);
        }
        this.angle = Math.atan2(ty - sy, tx - sx);
        this.speed = 4;
        this.acceleration = 1.05;
        this.brightness = random(50,70);
        this.targetRadius = 1;
      }
      update(index) {
        this.coordinates.pop();
        this.coordinates.unshift([this.x, this.y]);
        if(this.targetRadius < 8) {
          this.targetRadius += 0.3;
        } else {
          this.targetRadius = 1;
        }
        this.speed *= this.acceleration;
        let vx = Math.cos(this.angle) * this.speed;
        let vy = Math.sin(this.angle) * this.speed;
        this.distanceTraveled = getDistance(this.sx, this.sy, this.x + vx, this.y + vy);
        if(this.distanceTraveled >= this.distanceToTarget) {
          createParticles(this.tx, this.ty);
          fireworks.splice(index, 1);
        } else {
          this.x += vx;
          this.y += vy;
        }
      }
      draw() {
        ctx.beginPath();
        ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
        ctx.lineTo(this.x, this.y);
        ctx.strokeStyle = 'hsl(' + hue + ', 100%, ' + this.brightness + '%)';
        ctx.stroke();

        ctx.beginPath();
        ctx.arc(this.tx, this.ty, this.targetRadius, 0, Math.PI * 2);
        ctx.stroke();
      }
    }

    class Particle {
      constructor(x, y) {
        this.x = x;
        this.y = y;
        this.coordinates = [];
        this.coordinateCount = 5;
        while(this.coordinateCount--) {
          this.coordinates.push([this.x, this.y]);
        }
        this.angle = random(0, Math.PI * 2);
        this.speed = random(1, 10);
        this.friction = 0.95;
        this.gravity = 0.3;
        this.hue = random(hue - 50, hue + 50);
        this.brightness = random(50, 80);
        this.alpha = 1;
        this.decay = random(0.015, 0.03);
      }
      update(index) {
        this.coordinates.pop();
        this.coordinates.unshift([this.x, this.y]);
        this.speed *= this.friction;
        this.x += Math.cos(this.angle) * this.speed;
        this.y += Math.sin(this.angle) * this.speed + this.gravity;
        this.alpha -= this.decay;

        if(this.alpha <= this.decay) {
          particles.splice(index, 1);
        }
      }
      draw() {
        ctx.beginPath();
        ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
        ctx.lineTo(this.x, this.y);
        ctx.strokeStyle = 'hsla(' + this.hue + ', 100%, ' + this.brightness + '%, ' + this.alpha + ')';
        ctx.stroke();
      }
    }

    function random(min, max) {
      return Math.random() * (max - min) + min;
    }

    function getDistance(x1, y1, x2, y2) {
      const xDistance = x2 - x1;
      const yDistance = y2 - y1;
      return Math.sqrt(xDistance * xDistance + yDistance * yDistance);
    }

    function createParticles(x, y) {
      let particleCount = 30;
      while(particleCount--) {
        particles.push(new Particle(x, y));
      }
    }

    // Main animation loop
    function loop() {
      requestAnimationFrame(loop);
      ctx.globalCompositeOperation = 'destination-out';
      ctx.fillStyle = 'rgba(0, 0, 0, 0.25)';
      ctx.fillRect(0, 0, cw, ch);
      ctx.globalCompositeOperation = 'lighter';

      let i = fireworks.length;
      while(i--) {
        fireworks[i].draw();
        fireworks[i].update(i);
      }

      let j = particles.length;
      while(j--) {
        particles[j].draw();
        particles[j].update(j);
      }

      if(limiterTick >= limiterTotal) {
        if(timerTick >= timerTotal) {
          for(let k=0; k<simultaneousFireworks; k++) {
            let startX = cw / 4 + Math.random() * cw / 2;
            let startY = ch;
            let targetX = Math.random() * cw;
            let targetY = Math.random() * ch / 2;
            fireworks.push(new Firework(startX, startY, targetX, targetY));
          }
          timerTick = 0;
        } else {
          timerTick++;
        }
        limiterTick = 0;
      } else {
        limiterTick++;
      }
    }
    loop();

    // Letter by letter colorful text animation with fixed colors sequence
    const birthdayText = 'Happy Birthday pikachuuu!';
    const container = document.getElementById('birthday-text');
    container.innerHTML = '';
    // Fixed color pattern for letters in order
    const fixedColors = ['#ff4b2b', '#ff9a76', '#ff416c', '#fcbad3', '#ff6f91', // H A P P Y
                         '#00efeb', '#fff261', '#ff4e4e', '#ff7b72', '#4effe0', // B I R T H
                         '#ff69b4', '#ff1493', '#db7093', '#fff', '#ff4b2b']; // D A Y ! (plus extra for safe length)

    const animationDuration = 0.6; // seconds

    birthdayText.split('').forEach((char, i) => {
      const span = document.createElement('span');
      span.textContent = char;
      // assign fixed color or fallback to white
      span.style.color = fixedColors[i] || '#fff';
      span.style.animation = `letterAppear ${animationDuration}s ease forwards`;
      span.style.animationDelay = (i * animationDuration * 0.6) + 's';
      container.appendChild(span);
    });

    const style = document.createElement('style');
    style.textContent = `
      @keyframes letterAppear {
        0% {
          opacity: 0;
          transform: translateY(20px);
        }
        100% {
          opacity: 1;
          transform: translateY(0);
        }
      }
    `;
    document.head.appendChild(style);

    // Balloon creation
    const balloonColors = ['#ff4b2b', '#ff416c', '#ff6f91', '#fcbad3', '#ff9a76'];
    const balloonsCount = 15;

    function createBalloon(index) {
      const balloon = document.createElement('div');
      balloon.className = 'balloon';
      const color = balloonColors[index % balloonColors.length];
      balloon.style.background = color;
      balloon.style.left = Math.random() * 90 + 'vw';
      balloon.style.animationDuration = (8 + Math.random() * 5) + 's';
      balloon.style.animationDelay = (Math.random() * 5) + 's';
      const scale = 0.7 + Math.random() * 0.6;
      balloon.style.transform = `scale(${scale})`;
      document.body.appendChild(balloon);

      balloon.addEventListener('animationend', () => {
        balloon.remove();
        createBalloon(index);
      });
    }

    for (let i = 0; i < balloonsCount; i++) {
      createBalloon(i);
    }

    // Flower creation
    const flowerColors = ['#ffb6c1', '#ffc0cb', '#ff69b4', '#ff1493', '#db7093'];
    const flowersCount = 12;

    function createFlower(index) {
      const flower = document.createElement('div');
      flower.className = 'flower';

      for (let i = 1; i <= 4; i++) {
        const petal = document.createElement('div');
        petal.className = 'petal petal' + i;
        const petalColor = flowerColors[index % flowerColors.length];
        petal.style.backgroundColor = petalColor;
        flower.appendChild(petal);
      }

      const center = document.createElement('div');
      center.className = 'center';
      flower.appendChild(center);

      flower.style.left = Math.random() * 90 + 'vw';
      flower.style.animationDuration = (8 + Math.random() * 5) + 's';
      flower.style.animationDelay = (Math.random() * 5) + 's';

      const scale = 0.5 + Math.random() * 0.7;
      flower.style.transform = `scale(${scale})`;
      document.body.appendChild(flower);

      flower.addEventListener('animationend', () => {
        flower.remove();
        createFlower(index);
      });
    }

    for (let i = 0; i < flowersCount; i++) {
      createFlower(i);
    }
  </script>
</body>
</html>

