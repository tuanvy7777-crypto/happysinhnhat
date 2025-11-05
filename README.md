<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Thiệp Sinh Nhật - Nhấn để hiện quà</title>
  <style>
    :root{
      --pink:#ffd6e8;
      --deep-pink:#ff61a6;
      --text:#4a2a3a;
    }
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{
      background: linear-gradient(180deg, var(--pink) 0%, #ffeef8 100%);
      display:flex;align-items:center;justify-content:center;padding:20px;
      overflow:hidden;
    }
    .stage{
      width:100%;max-width:900px;height:600px;border-radius:16px;box-shadow:0 10px 40px rgba(72,20,60,0.12);
      background: radial-gradient(1200px 400px at 10% 10%, rgba(255,255,255,0.6), transparent 10%), rgba(255,245,250,0.6);
      position:relative;display:flex;align-items:center;justify-content:center;flex-direction:column;padding:24px;gap:16px;
    }

    /* button */
    .open-btn{
      background:linear-gradient(90deg,var(--deep-pink),#ff8ccf);
      color:white;border:none;padding:18px 28px;border-radius:999px;font-weight:700;font-size:18px;cursor:pointer;
      box-shadow:0 6px 16px rgba(255,92,156,0.28);transition:transform .16s ease,box-shadow .16s;
    }
    .open-btn:active{transform:translateY(2px);box-shadow:0 3px 8px rgba(255,92,156,0.18)}

    /* hidden content */
    .content{display:none;align-items:center;gap:20px;text-align:center}
    .images{display:flex;gap:18px;flex-wrap:wrap;align-items:center;justify-content:center}
    .images img{width:260px;max-width:42vw;border-radius:12px;box-shadow:0 10px 30px rgba(80,10,60,0.12);transform:translateY(20px);opacity:0;transition:all .7s cubic-bezier(.2,.9,.3,1)}
    .images img.show{transform:none;opacity:1}

    .message{
      font-size:26px;color:var(--text);font-weight:700;padding:12px 18px;border-radius:12px;background:linear-gradient(90deg, rgba(255,255,255,0.6), rgba(255,255,255,0.25));backdrop-filter:blur(6px);
      transform:translateY(20px);opacity:0;transition:all .7s .1s cubic-bezier(.2,.9,.3,1)
    }
    .message.show{transform:none;opacity:1}

    /* little sparkle under message */
    .message small{display:block;font-weight:500;font-size:14px;margin-top:6px;color:#6b3b4f}

    /* decorative hearts/flower dots */
    .decor{position:absolute;inset:0;pointer-events:none}

    /* responsive */
    @media (max-width:640px){
      .stage{height:90vh;padding:14px}
      .images img{width:44vw}
      .message{font-size:20px}
      .open-btn{padding:14px 20px;font-size:16px}
    }
  </style>
</head>
<body>
  <div class="stage" role="main" aria-live="polite">
    <button class="open-btn" id="openBtn">Nhấn vô nha🎁</button>

    <div class="content" id="content">
      <div class="images" id="images">
        <img id="img1" alt="Hình 1" src="img1.jpg">
        <img id="img2" alt="Hình 2" src="img2.jpg">
      </div>
      <div class="message" id="message">
        Chúc mừng sinh nhật em. Chúc em tuổi mới đầy niềm vui và hạnh phúc!
        Chúc em có tất cả trừ vất vả.
        <small>Hôm nay là của em nên hãy luôn mỉm cười nhé em 🎉</small>
      </div>
    </div>

    <!-- falling flower canvas -->
    <canvas id="fallCanvas" class="decor"></canvas>
  </div>

  <script>
    // Elements
    const openBtn = document.getElementById('openBtn');
    const content = document.getElementById('content');
    const images = document.querySelectorAll('.images img');
    const message = document.getElementById('message');
    const canvas = document.getElementById('fallCanvas');
    const ctx = canvas.getContext('2d');

    // Make canvas full area of stage
    function resizeCanvas(){
      const rect = canvas.getBoundingClientRect();
      canvas.width = rect.width;
      canvas.height = rect.height;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // simple flower particle using emoji drawn on canvas
    class Petal{
      constructor(){
        this.reset();
      }
      reset(){
        this.x = Math.random() * canvas.width;
        this.y = -20 - Math.random()*canvas.height*0.2;
        this.size = 14 + Math.random()*20;
        this.speedY = 0.5 + Math.random()*2.2;
        this.speedX = Math.random()*1.6 - 0.8;
        this.rotation = Math.random()*360;
        this.spin = (Math.random()*2-1)*0.02;
        // pick an emoji flower variation
        const choices = ['🌸','🌺','💮','🌷','🌼'];
        this.ch = choices[Math.floor(Math.random()*choices.length)];
        this.alpha = 0.8 - Math.random()*0.5;
      }
      update(){
        this.y += this.speedY;
        this.x += this.speedX + Math.sin(this.y/40)*0.6;
        this.rotation += this.spin;
        if(this.y > canvas.height + 50) this.reset();
      }
      draw(ctx){
        ctx.save();
        ctx.globalAlpha = this.alpha;
        ctx.translate(this.x, this.y);
        ctx.rotate(this.rotation);
        ctx.font = Math.round(this.size) + 'px serif';
        ctx.textAlign = 'center';
        ctx.fillText(this.ch, 0, 0);
        ctx.restore();
      }
    }

    let petals = [];
    function initPetals(n=40){
      petals = [];
      for(let i=0;i<n;i++) petals.push(new Petal());
    }

    let animId;
    function animate(){
      ctx.clearRect(0,0,canvas.width,canvas.height);
      for(const p of petals){
        p.update();
        p.draw(ctx);
      }
      animId = requestAnimationFrame(animate);
    }

    // Reveal sequence when pressing button
    openBtn.addEventListener('click', ()=>{
      // disable button and animate
      openBtn.disabled = true;
      openBtn.style.transform = 'scale(.98)';

      // show content container
      content.style.display = 'flex';

      // stagger images
      images.forEach((img,i)=>{
        setTimeout(()=>{
          img.classList.add('show');
        }, 280 + i*240);
      });

      // show message after images
      setTimeout(()=>{
        message.classList.add('show');
      }, 800);

      // start particle effect
      resizeCanvas();
      initPetals(60);
      if(animId) cancelAnimationFrame(animId);
      animate();

      // small confetti burst from center using DOM elements
      burstConfetti();
    });

    // Extra: quick DOM confetti burst
    function burstConfetti(){
      const parent = document.querySelector('.stage');
      for(let i=0;i<24;i++){
        const el = document.createElement('div');
        el.style.position='absolute';
        el.style.left='50%';
        el.style.top='45%';
        el.style.width='10px';
        el.style.height='14px';
        el.style.opacity='0.95';
        el.style.transformOrigin='center';
        el.style.borderRadius='3px';
        el.style.pointerEvents='none';
        // random pastel color
        const cols = ['#ffd6e8','#ffb3d6','#ffe9f2','#ffe9d6','#ffd6f2','#fff0f6'];
        el.style.background = cols[Math.floor(Math.random()*cols.length)];
        parent.appendChild(el);
        // animate with JS
        const dx = (Math.random()*2-1)*260;
        const dy = -200 - Math.random()*160;
        const rot = (Math.random()*2-1)*720;
        el.animate([
          {transform:'translate(-50%,-50%) translate(0,0) rotate(0deg)',opacity:1},
          {transform:`translate(-50%,-50%) translate(${dx}px, ${dy}px) rotate(${rot}deg)`,opacity:0}
        ],{duration:1200 + Math.random()*600, easing:'cubic-bezier(.2,.8,.2,1)'}).onfinish = ()=>el.remove();
      }
    }

    // handle visibility if user resizes before clicking
    window.addEventListener('load', resizeCanvas);
  </script>
</body>
</html>
