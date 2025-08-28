<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Um Pedido para a Lua</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Montserrat', sans-serif;
      background: linear-gradient(135deg, #f5e1ee, #e1c4f5);
      overflow: hidden;
      cursor: default;
    }
    h1 {
      font-family: 'Playfair Display', serif;
    }
    #noBtn {
      position: absolute;
      transition: top 0.5s ease-in-out, left 0.5s ease-in-out;
    }
    .star {
      position: absolute;
      font-size: 1.5rem;
      color: #ffd700;
      pointer-events: none;
      animation: star-fade 1s forwards;
    }
    @keyframes star-fade {
      0% { opacity: 1; transform: translateY(0) scale(1); }
      100% { opacity: 0; transform: translateY(-50px) scale(0.5); }
    }
    /* Adjust button container for mobile */
    @media (max-width: 768px) {
      #buttonContainer {
        position: relative;
        height: 250px; /* Increased for more movement space */
      }
      #yesBtn {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 10px;
      }
      #noBtn {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 100px; /* Initial position further down */
      }
    }
  </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">
  <div id="proposal" class="p-6 md:p-8 bg-white/80 backdrop-blur-sm rounded-2xl shadow-2xl max-w-md w-full relative text-center" style="height: 350px;">
    <h1 class="text-4xl md:text-5xl text-pink-700 mb-8">Lua, quer iluminar minha vida pra sempre?</h1>
    <div id="buttonContainer" class="relative h-24 md:h-24">
      <button id="yesBtn" class="absolute left-1/2 -translate-x-1/2 md:left-1/4 md:-translate-x-1/2 px-8 py-4 w-auto text-lg font-bold bg-pink-500 text-white rounded-full hover:bg-pink-600 transition transform hover:scale-110 shadow-lg">Com toda certeza!</button>
      <button id="noBtn" class="absolute left-1/2 -translate-x-1/2 md:left-3/4 md:-translate-x-1/2 px-8 py-4 w-auto text-lg font-bold bg-gray-300 text-gray-800 rounded-full hover:bg-gray-400 transition shadow-lg">Não</button>
    </div>
  </div>
  <div id="confirmation" class="hidden p-6 md:p-8 bg-white/80 backdrop-blur-sm rounded-2xl shadow-2xl max-w-md w-full text-center">
    <h1 class="text-4xl md:text-5xl text-pink-700 mb-4">Eu já sabia!</h1>
    <p class="text-lg md:text-xl text-gray-700 mb-6">Obrigado por ser a luz da minha vida. Eu te amo.</p>
    <div class="text-pink-500 text-6xl animate-pulse">💍</div>
  </div>
  <div id="areYouSureModal" class="hidden p-6 md:p-8 bg-white/80 backdrop-blur-sm rounded-2xl shadow-2xl max-w-md w-full text-center">
    <h1 id="modalQuestion" class="text-4xl md:text-5xl text-pink-700 mb-8">Tem certeza?</h1>
    <div class="flex justify-center space-x-6">
      <button id="confirmYesBtn" class="px-8 py-4 text-lg font-bold bg-pink-500 text-white rounded-full hover:bg-pink-600 transition transform hover:scale-110 shadow-lg">Sim</button>
      <button id="confirmNoBtn" class="px-8 py-4 text-lg font-bold bg-gray-300 text-gray-800 rounded-full hover:bg-gray-400 transition shadow-lg">Não</button>
    </div>
  </div>
  <script>
    const proposal = document.getElementById('proposal');
    const confirmation = document.getElementById('confirmation');
    const areYouSureModal = document.getElementById('areYouSureModal');
    const yesBtn = document.getElementById('yesBtn');
    const noBtn = document.getElementById('noBtn');
    const buttonContainer = document.getElementById('buttonContainer');
    const modalQuestion = document.getElementById('modalQuestion');
    const confirmYesBtn = document.getElementById('confirmYesBtn');
    const confirmNoBtn = document.getElementById('confirmNoBtn');
    let wanderInterval;
    let certezaCount = 0;

    // Star effect
    const createStar = (x, y) => {
      const star = document.createElement('div');
      star.className = 'star';
      star.innerHTML = '✨';
      star.style.left = x + 'px';
      star.style.top = y + 'px';
      document.body.appendChild(star);
      setTimeout(() => star.remove(), 1000);
    };
    document.addEventListener('mousemove', (e) => createStar(e.pageX, e.pageY));
    document.addEventListener('touchmove', (e) => {
      if (e.touches.length > 0) {
        const touch = e.touches[0];
        createStar(touch.pageX, touch.pageY);
      }
    });

    // Move "Não" button with smoother, directional movement
    let currentX = null;
    let currentY = null;
    const moveButton = () => {
      if (proposal.classList.contains('hidden')) return;
      const containerRect = buttonContainer.getBoundingClientRect();
      const yesBtnRect = yesBtn.getBoundingClientRect();
      const noBtnRect = noBtn.getBoundingClientRect();
      const maxX = containerRect.width - noBtnRect.width;
      const maxY = containerRect.height - noBtnRect.height;
      const minY = window.innerWidth <= 768 ? 80 : 0; // Restrict "Não" to below "Sim" on mobile

      // Initialize current position if not set
      if (currentX === null || currentY === null) {
        currentX = parseFloat(noBtn.style.left) || maxX / 2;
        currentY = parseFloat(noBtn.style.top) || (window.innerWidth <= 768 ? 100 : maxY / 2);
      }

      // Generate random direction (angle in radians)
      const angle = Math.random() * 2 * Math.PI;
      const speed = window.innerWidth <= 768 ? 25 : 40; // Smaller steps on mobile
      const deltaX = Math.cos(angle) * speed;
      const deltaY = Math.sin(angle) * speed;

      // Calculate new position
      let newX = currentX + deltaX;
      let newY = currentY + deltaY;

      // Keep within bounds
      newX = Math.max(0, Math.min(newX, maxX));
      newY = Math.max(minY, Math.min(newY, maxY));

      // Avoid "Com toda certeza" button
      const noBtnCenterX = newX + noBtnRect.width / 2;
      const noBtnCenterY = newY + noBtnRect.height / 2;
      const yesBtnCenterX = yesBtnRect.left - containerRect.left + yesBtnRect.width / 2;
      const yesBtnCenterY = yesBtnRect.top - containerRect.top + yesBtnRect.height / 2;
      const minDistance = window.innerWidth <= 768 ? 80 : 100; // Stricter on mobile
      const distance = Math.sqrt(
        Math.pow(noBtnCenterX - yesBtnCenterX, 2) + Math.pow(noBtnCenterY - yesBtnCenterY, 2)
      );

      // Only update position if not too close to "Sim" button
      if (distance > minDistance) {
        noBtn.style.left = newX + 'px';
        noBtn.style.top = newY + 'px';
        currentX = newX;
        currentY = newY;
      } else {
        // Try a different direction if too close
        moveButton();
      }
    };

    const startWandering = () => {
      stopWandering();
      wanderInterval = setInterval(moveButton, 800); // Move every 0.8s
    };

    const stopWandering = () => {
      clearInterval(wanderInterval);
    };

    const panicMove = () => {
      stopWandering();
      noBtn.style.transition = 'top 0.2s ease, left 0.2s ease';
      moveButton();
      setTimeout(() => {
        noBtn.style.transition = 'top 0.5s ease-in-out, left 0.5s ease-in-out';
        startWandering();
      }, 800);
    };

    // Handle "Não" button interaction
    const handleNoInteraction = () => {
      stopWandering();
      certezaCount = 0;
      updateModalQuestion();
      proposal.classList.add('hidden');
      areYouSureModal.classList.remove('hidden');
    };

    noBtn.addEventListener('click', handleNoInteraction);
    noBtn.addEventListener('touchstart', (e) => {
      e.preventDefault();
      handleNoInteraction();
      panicMove();
    });
    noBtn.addEventListener('mouseenter', panicMove);

    yesBtn.addEventListener('click', () => {
      stopWandering();
      proposal.classList.add('hidden');
      areYouSureModal.classList.add('hidden');
      confirmation.classList.remove('hidden');
      confetti({ particleCount: 200, spread: 90, origin: { y: 0.6 } });
    });

    confirmNoBtn.addEventListener('click', () => {
      areYouSureModal.classList.add('hidden');
      proposal.classList.remove('hidden');
      startWandering();
    });

    confirmYesBtn.addEventListener('click', () => {
      certezaCount++;
      updateModalQuestion();
    });

    function updateModalQuestion() {
      let message = "Certeza";
      if (certezaCount > 0) message += " mesmo".repeat(certezaCount);
      message += "?";
      modalQuestion.textContent = message;
    }

    // Start wandering on page load
    startWandering();
  </script>
</body>
</html>
