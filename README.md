<!DOCTYPE html>
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
      position: absolute; /* Essencial para o movimento funcionar */
      transition: top 1.5s ease-in-out, left 1.5s ease-in-out;
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
  </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">
  <div id="proposal" class="p-6 md:p-8 bg-white/80 backdrop-blur-sm rounded-2xl shadow-2xl max-w-md w-full relative text-center" style="height: 300px;"> <h1 class="text-4xl md:text-5xl text-pink-700 mb-8">Lua, quer iluminar minha vida pra sempre?</h1>
    <div id="buttonContainer" class="relative h-24"> <button id="yesBtn" class="absolute left-1/2 -translate-x-1/2 md:left-1/4 md:-translate-x-1/2 px-8 py-4 w-auto text-lg font-bold bg-pink-500 text-white rounded-full hover:bg-pink-600 transition transform hover:scale-110 shadow-lg">Com toda certeza!</button>
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
    // --- Configuração de TODOS os Elementos ---
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

    // --- Lógica das Estrelas ---
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

    // --- Lógica do Botão "Não" com Patrulha e Pânico ---
    const moveButton = () => {
      if (proposal.classList.contains('hidden')) return;
      const containerRect = buttonContainer.getBoundingClientRect();
      const proposalRect = proposal.getBoundingClientRect();
      const noBtnRect = noBtn.getBoundingClientRect();
      
      const maxX = containerRect.width - noBtnRect.width;
      const maxY = containerRect.height - noBtnRect.height;
      
      noBtn.style.left = (Math.random() * maxX) + 'px';
      noBtn.style.top = (Math.random() * maxY) + 'px';
    };

    const startWandering = () => {
      stopWandering();
      wanderInterval = setInterval(moveButton, 2000);
    };

    const stopWandering = () => {
      clearInterval(wanderInterval);
    };
    
    const panicMove = () => {
      stopWandering();
      noBtn.style.transition = 'top 0.2s ease, left 0.2s ease';
      moveButton();
      setTimeout(() => {
        noBtn.style.transition = 'top 1.5s ease-in-out, left 1.5s ease-in-out';
        startWandering();
      }, 2500);
    };
    noBtn.addEventListener('mouseenter', panicMove);
    noBtn.addEventListener('touchstart', (e) => { e.preventDefault(); panicMove(); });
    
    // --- Lógica Principal dos Cliques ---
    let certezaCount = 0;

    yesBtn.addEventListener('click', () => {
      stopWandering();
      proposal.classList.add('hidden');
      areYouSureModal.classList.add('hidden');
      confirmation.classList.remove('hidden');
      confetti({ particleCount: 200, spread: 90, origin: { y: 0.6 } });
    });

    noBtn.addEventListener('click', () => {
      stopWandering();
      certezaCount = 0;
      updateModalQuestion();
      proposal.classList.add('hidden');
      areYouSureModal.classList.remove('hidden');
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
    
    // Inicia a patrulha assim que a página carrega
    startWandering();
  </script>
</body>
</html>
