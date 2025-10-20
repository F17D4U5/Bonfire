<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Bonfire: Core Survival</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: sans-serif; background-color: #1e293b; color: #f8fafc; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: 0 auto; }
        .night-mode { background-color: #1a202c !important; color: #a5b4fc !important; }
        #action-panel { transition: background-color 2s ease-in-out; }
        .night-mode #action-panel { background-color: #0f172a / 80; }
        .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 20px; }
        .stat-box { 
            background-color: #334155; 
            padding: 10px; 
            border-radius: 6px; 
            transition: background-color 0.5s ease;
        }
        .time-status { font-weight: bold; font-size: 1.25rem; }
        .night-mode { background-color: #1a202c; color: #a5b4fc; }
        .night-mode .stat-box { background-color: #1e293b; } /* Efek malam pada stat box */
        .log-error { color: #f87171; font-weight: bold; }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-3xl font-bold mb-4">🔥 The Bonfire: Core Survival</h1>
        
        <div id="game-status" class="flex justify-between items-center bg-sky-900 p-3 rounded-lg mb-4">
            <span id="time-display" class="time-status"></span>
            <span id="log-display" class="text-sm italic">Selamat datang di pantai tak dikenal.</span>
        </div>

        <div class="stats-grid">
            <div class="stat-box">🪵 Kayu: <span id="wood-stat">0</span></div>
            <div class="stat-box">🥩 Makanan: <span id="food-stat">0</span></div>
            <div class="stat-box">👨‍👩‍👧 Penduduk: <span id="population-stat">1</span></div>
        </div>

        <div id="canvas-wrapper" class="relative bg-slate-800 p-0 rounded-lg mb-4 flex justify-center">
            
            <canvas id="gameCanvas" width="700" height="300" class="bg-gray-900 border border-gray-600 rounded-md"></canvas>

            <div id="action-panel" class="absolute top-4 left-4 p-3 bg-slate-700/80 rounded-lg backdrop-blur-sm z-10 shadow-lg">
                <h2 class="text-lg font-semibold mb-2 text-white border-b border-slate-500 pb-1">Alokasi Pekerja</h2>
                
                <div class="flex flex-col gap-3">
                    
                    <div class="flex justify-between items-center">
                        <span class="text-sm">🪵 Pengumpul Kayu: <span id="gatherer-stat-short">0</span></span>
                        <div>
                            <button onclick="modifyWorker('gatherer', -1)" class="bg-gray-500 hover:bg-gray-600 text-white w-6 h-6 rounded-l disabled:opacity-50 text-sm font-bold">-</button>
                            <button onclick="modifyWorker('gatherer', 1)" class="bg-green-600 hover:bg-green-700 text-white w-6 h-6 rounded-r disabled:opacity-50 text-sm font-bold">+</button>
                        </div>
                    </div>
                    
                    <div class="flex justify-between items-center">
                        <span class="text-sm">🥩 Hunter: <span id="hunter-stat-short">0</span> / <span id="max-hunter-stat-short">0</span></span>
                        <div>
                            <button onclick="modifyWorker('hunter', -1)" class="bg-gray-500 hover:bg-gray-600 text-white w-6 h-6 rounded-l disabled:opacity-50 text-sm font-bold">-</button>
                            <button onclick="modifyWorker('hunter', 1)" class="bg-yellow-600 hover:bg-yellow-700 text-white w-6 h-6 rounded-r disabled:opacity-50 text-sm font-bold">+</button>
                        </div>
                    </div>
                    
                    <div class="flex justify-between items-center">
                        <span class="text-sm">🛡️ Penjaga: <span id="guard-stat-short">0</span></span>
                        <div>
                            <button onclick="modifyWorker('guard', -1)" class="bg-gray-500 hover:bg-gray-600 text-white w-6 h-6 rounded-l disabled:opacity-50 text-sm font-bold">-</button>
                            <button onclick="modifyWorker('guard', 1)" class="bg-red-600 hover:bg-red-700 text-white w-6 h-6 rounded-r disabled:opacity-50 text-sm font-bold">+</button>
                        </div>
                    </div>
                </div>
                
                <p class="mt-4 text-xs text-white/80 border-t border-slate-500 pt-2">Pekerja Bebas: <span id="free-workers-stat-short">1</span></p>
            </div>
            <div class="bg-yellow-800 p-3 rounded-lg absolute top-4 right-4 z-10 shadow-lg" id="wanderer-panel" style="display: none;">
                <h3 class="text-lg font-semibold mb-2">Seorang Pengembara Tiba!</h3>
                <p class="text-sm mb-3">Orang ini kelaparan dan meminta <span id="wanderer-cost">3</span>🥩 Makanan untuk menetap.</p>
                <div class="flex gap-4">
                    <button onclick="handleWanderer('accept')" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-2 px-4 rounded disabled:opacity-50" id="wanderer-accept-btn">
                        Terima & Beri Makan
                    </button>
                    <button onclick="handleWanderer('reject')" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded" id="wanderer-reject-btn">
                        Tolak
                    </button>
                </div>
            </div>

        </div>
        
        <div class="bg-slate-700 p-4 rounded-lg">
            <h2 class="text-xl font-semibold mb-3">Pembangunan</h2>
            <button onclick="buildBuilding('hunterHut')" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded disabled:opacity-50" id="build-hunterhut-btn">
                Bangun Hunter Hut (20 🪵) (<span id="hunterhut-count">0</span>)
            </button>
            
            <button onclick="buildBuilding('storageHut')" class="bg-gray-600 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded disabled:opacity-50 ml-4" id="build-storagehut-btn">
                Bangun Storage Hut (30 🪵) (<span id="storagehut-count">0</span>)
            </button>
            
        </div>
        
    </div>

    <div id="game-over-overlay" class="fixed inset-0 bg-black bg-opacity-80 flex flex-col justify-center items-center z-50" style="display: none;">
        <h1 class="text-6xl font-extrabold text-red-600 mb-4 animate-pulse">GAME OVER</h1>
        <p id="game-over-cause" class="text-xl text-white mb-8 text-center px-4">Desa Anda hancur dan tidak ada penduduk yang tersisa.</p>
        <button onclick="restartGame()" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg text-lg transition duration-300 transform hover:scale-105">
            Mulai Ulang (Hari 1)
        </button>
    </div>

    <script>
// --- DEKLARASI STATE GAME ---
const initialGameState = {
    population: 1, freeWorkers: 1, maxWorkers: 1, 
    resources: { wood: 100, food: 50, }, 
    storage: { woodMax: 100, foodMax: 100, baseCapacity: 100, capacityPerHut: 100, },
    time: { dayCount: 1, hour: 8, isNight: false, isAttackPhase: false, },
    workers: { gatherer: 0, guard: 0, hunter: 0, },
    buildings: { 
        hunterHut: { count: 0, workers: 0, cost: { wood: 20 }, maxWorkers: 1, }, 
        storageHut: { count: 0, workers: 0, cost: { wood: 30 }, maxWorkers: 0, }, 
    },
    defense: { raidStrength: 0, },
    events: { isWandererPresent: false, wandererFoodCost: 3, wandererArrivalHour: 0, },
    bonfire: { woodCostPerCycle: 1, isBurning: true, },
    isGameOver: false, 
};

let gameState = JSON.parse(JSON.stringify(initialGameState));

// Variabel DOM
const timeDisplay = document.getElementById('time-display');
const logDisplay = document.getElementById('log-display');
const container = document.querySelector('.container');
let canvas;
let ctx;
let animationFrameId; 
let BONFIRE_POS;
let WORKER_START_Y;
let timeIntervalId;
let cityIntervalId;

// Konstanta
const TILE_SIZE = 50; 
const WORKER_COLORS = { 
    gatherer: '#10b981', 
    hunter: '#3b82f6', 
    guard: '#ef4444', 
    freeWorker: '#facc15', 
    raider: '#4b5563', 
};

// --- VARIABEL BARU UNTUK GERAK ACAK PEKERJA ---
let visualWorkers = [];
const REST_RADIUS = 70; // Jarak maksimal dari pusat istirahat
const MIN_DISTANCE_FROM_BONFIRE = 15; // Jarak minimal dari api unggun (pusat BONFIRE_POS)
const TRANSITION_SPEED = 6.0; // Kecepatan lari pekerja saat transisi

/* -------------- HELPER & GAME OVER -------------- */
function gameLog(message, isError = false) {
    logDisplay.textContent = message;
    logDisplay.classList.toggle('log-error', isError);
}

function interpolateColor(color1, color2, factor) {
    if (factor < 0) factor = 0;
    if (factor > 1) factor = 1;
    
    const hexToRgb = (hex) => [
        parseInt(hex.substring(1, 3), 16),
        parseInt(hex.substring(3, 5), 16),
        parseInt(hex.substring(5, 7), 16),
    ];
    
    const rgb1 = hexToRgb(color1);
    const rgb2 = hexToRgb(color2);
    
    const r = Math.round(rgb1[0] + (rgb2[0] - rgb1[0]) * factor);
    const g = Math.round(rgb1[1] + (rgb2[1] - rgb1[1]) * factor);
    const b = Math.round(rgb1[2] + (rgb2[2] - rgb1[2]) * factor);
    
    return `rgb(${r}, ${g}, ${b})`;
}

function endGame(cause) {
    if (gameState.isGameOver) return; 

    gameState.isGameOver = true;
    
    clearInterval(timeIntervalId);
    clearInterval(cityIntervalId);
    cancelAnimationFrame(animationFrameId);

    const gameOverOverlay = document.getElementById('game-over-overlay');
    const causeMessage = document.getElementById('game-over-cause');
    
    if (cause === "kelaparan") {
        causeMessage.textContent = "Kekalahan! Semua penduduk tewas kelaparan. Kelola makanan dengan lebih baik.";
    } else if (cause === "serangan") {
        causeMessage.textContent = "Kekalahan! Desa hancur dan penduduk musnah setelah serangan yang brutal.";
    } else {
        causeMessage.textContent = "GAME OVER: Semua penduduk hilang.";
    }
    
    gameOverOverlay.style.display = 'flex';
    gameLog("!!! GAME OVER !!!", true);
}

function restartGame() {
    clearInterval(timeIntervalId);
    clearInterval(cityIntervalId);
    
    gameState = JSON.parse(JSON.stringify(initialGameState));
    visualWorkers = []; // Reset visual workers
    
    document.getElementById('game-over-overlay').style.display = 'none';
    
    container.classList.remove('night-mode');
    
    init(); 
    gameLog("Game diulang! Selamat datang kembali, Survivor.");
}

/* -------------- A. MEKANIK WAKTU & SIKLUS (PENAMBAHAN LOGIKA TRANSISI) -------------- */
function updateTime() {
    if (gameState.isGameOver) return; 
    
    gameState.time.hour++;

    if (gameState.time.hour >= 24) {
        gameState.time.hour = 0;
        gameState.time.dayCount++;
        gameLog(`Hari ${gameState.time.dayCount} telah tiba!`);
        
        if (gameState.time.dayCount % 5 === 0) {
             gameState.population++;
             gameState.freeWorkers++;
             gameState.maxWorkers++;
             gameLog(`Wanderer baru bergabung! Populasi: ${gameState.population}`);
        }
    }

    const wasNight = gameState.time.isNight;
    const wasAttackPhase = gameState.time.isAttackPhase;
    
    if (!gameState.time.isNight && !gameState.events.isWandererPresent && gameState.time.dayCount > 1) {
        if (gameState.time.hour >= 9 && gameState.time.hour <= 12) {
            if (Math.random() < 0.10) { 
                gameState.events.isWandererPresent = true;
                gameState.events.wandererArrivalHour = gameState.time.hour;
                gameLog(`👋 Seorang Pengembara tiba! Klik tombol 'Terima Pengembara' di kanan atas.`);
            }
        }
    }

    if (gameState.events.isWandererPresent && gameState.time.hour === 15) {
        gameLog(`Pengembara itu pergi karena Anda tidak menawarkan makanan.`, true);
        gameState.events.isWandererPresent = false;
    }
    
    // 1. Fase Persiapan Malam (Jam 20:00)
    if (gameState.time.hour === 20 && !wasNight) {
        gameState.time.isNight = true; 
        gameLog("🔔 Matahari terbenam! Persiapkan penjaga Anda!");
        container.classList.add('night-mode');
        
        // *TRANSISI VISUAL: LARI PULANG*
        const REST_CENTER_Y = BONFIRE_POS.y + 40;
        visualWorkers.forEach(worker => {
            if (worker.type === 'gatherer' || worker.type === 'hunter') {
                worker.targetX = BONFIRE_POS.x + (Math.random() * 20 - 10); // sedikit acak
                worker.targetY = REST_CENTER_Y + (Math.random() * 20 - 10);
                worker.isTargetedMoving = true;
            }
        });
    }

    // 2. Transisi ke FASE SERANGAN (Jam 22:00)
    if (gameState.time.hour === 22 && !wasAttackPhase) {
        gameState.time.isAttackPhase = true;
        gameState.defense.raidStrength = gameState.time.dayCount * 2 + Math.floor(Math.random() * 5); 
        
        gameLog(`Musuh mendekat! Kekuatan serangan: ${gameState.defense.raidStrength}.`);
        handleNightRaid();
    }
    
    // 3. Transisi ke Siang (Jam 08:00)
    if (gameState.time.hour === 8 && wasNight) {
        gameState.time.isNight = false;
        gameState.time.isAttackPhase = false; 
        
        gameState.defense.raidStrength = 0;
        
        gameLog("Selamat pagi! Saatnya bekerja kembali.");
        container.classList.remove('night-mode');
        
        // *TRANSISI VISUAL: LARI KERJA*
        const WORK_SPOT_X = 50;
        const WORK_SPOT_Y = BONFIRE_POS.y;
        
        let hunterIndex = 0;
        let gathererIndex = 0;
        
        visualWorkers.forEach((worker, i) => {
            if (worker.type === 'gatherer') {
                const offset = 0; 
                worker.targetX = WORK_SPOT_X + (Math.random() * 10 - 5);
                worker.targetY = WORK_SPOT_Y + offset + (gathererIndex * 15) + (Math.random() * 10 - 5);
                worker.isTargetedMoving = true;
                gathererIndex++;
            } else if (worker.type === 'hunter') {
                 const offset = 30; 
                worker.targetX = WORK_SPOT_X + (Math.random() * 10 - 5);
                worker.targetY = WORK_SPOT_Y + offset + (hunterIndex * 15) + (Math.random() * 10 - 5);
                worker.isTargetedMoving = true;
                hunterIndex++;
            }
        });
    }
    
    updateUI();
}

/* -------------- B. MEKANIK UTAMA GAME (TIDAK ADA PERUBAHAN) -------------- */
function updateCity() {
    // 1. LOGIKA BANGUNAN
    if (gameState.isGameOver) return;
    
    const storageHutCount = gameState.buildings.storageHut.count;
    const bonusStorage = storageHutCount * gameState.storage.capacityPerHut;
    
    gameState.storage.woodMax = gameState.storage.baseCapacity + bonusStorage;
    gameState.storage.foodMax = gameState.storage.baseCapacity + bonusStorage;

    // 2. PENGUMPULAN SUMBER DAYA
    let woodGained = 0;
    let foodProduced = 0;
    
    // Pengumpul kayu dan Hunter HANYA BEKERJA saat TIDAK malam.
    if (!gameState.time.isNight) {
        woodGained = gameState.workers.gatherer * 3;
        foodProduced = gameState.workers.hunter * 5; 
    }
    
    gameState.resources.wood += woodGained;
    gameState.resources.food += foodProduced; 
    
    gameState.resources.wood = Math.min(gameState.resources.wood, gameState.storage.woodMax);
    gameState.resources.food = Math.min(gameState.resources.food, gameState.storage.foodMax);
    
    // 3. KONSUMSI MAKANAN
    gameState.resources.food -= gameState.population * 1; 

    // 4. LOGIKA BONFIRE: Konsumsi Kayu
    const cost = gameState.bonfire.woodCostPerCycle; 
    
    if (gameState.resources.wood >= cost) {
        gameState.resources.wood -= cost;
        gameState.bonfire.isBurning = true;
    } else {
        gameState.bonfire.isBurning = false;
        
        if (gameState.freeWorkers > 0) {
            gameState.freeWorkers = Math.max(0, gameState.freeWorkers - 1);
            gameLog(`🔥 API PADAM! Kita kehilangan semangat. 1 Pekerja menjadi tidak efisien.`, true);
        } else {
            gameLog(`🔥 API PADAM! Segera kumpulkan kayu!`, true);
        }
    }

    // 5. SURVIVAL CHECK: Kelaparan (Bisa menyebabkan Game Over)
    if (gameState.resources.food < 0) {
        gameState.resources.food = 0;
        if (Math.random() < 0.05 && gameState.population > 0) { 
            gameState.population = gameState.population - 1; 
            gameState.maxWorkers = gameState.population;
            gameState.freeWorkers = Math.max(0, gameState.freeWorkers - 1); 
            gameLog("Penduduk meninggal karena kelaparan! Kelola makanan dengan baik.", true);
        }
    }
    
    // 6. GAME OVER CHECK
    if (gameState.population <= 0) {
        endGame("kelaparan");
    }

    updateUI();
}

/* -------------- C. MEKANIK PERTAHANAN MALAM (TIDAK ADA PERUBAHAN) -------------- */
function handleNightRaid() {
    const attack = gameState.defense.raidStrength;
    
    let defensePower = gameState.workers.guard * 5; 
    if (!gameState.bonfire.isBurning) {
        defensePower = Math.floor(defensePower / 2); 
        gameLog(`Api padam! Pertahanan melemah menjadi ${defensePower}.`, true);
    }
    
    if (attack === 0) return; 

    if (defensePower >= attack) {
        gameState.resources.wood += 5;
        gameLog(`🎉 Kemenangan! Penjaga berhasil mengusir mereka dan mengamankan 5 Kayu tambahan!`);
    } else {
        const damageTaken = attack - defensePower;
        gameState.resources.wood = Math.max(0, gameState.resources.wood - damageTaken * 3);
        
        if (damageTaken > 10 && gameState.population > 0) { 
            gameState.population = gameState.population - 1; 
            gameState.maxWorkers = gameState.population;
            
            if (gameState.workers.guard > 0) {
                gameState.workers.guard = Math.max(0, gameState.workers.guard - 1);
                gameLog(`💀 SERANGAN GAGAL TOTAL! Kita kehilangan 1 penduduk (Penjaga) dan Kayu akibat serangan sebesar ${damageTaken}.`, true);
            } else {
                 gameState.freeWorkers = Math.max(0, gameState.freeWorkers - 1); 
                 gameLog(`💀 SERANGAN GAGAL TOTAL! Kita kehilangan 1 penduduk dan Kayu akibat serangan sebesar ${damageTaken}.`, true);
            }

        } else {
            gameLog(`Serangan berhasil dipukul mundur, tapi kita kehilangan Kayu. Kerugian: ${damageTaken}.`);
        }
    }
    
    if (gameState.population <= 0) {
        endGame("serangan");
        return; 
    }
    
    gameState.defense.raidStrength = 0; 
    updateUI();
}

/* -------------- D. LOGIKA PEKERJA & BANGUNAN (TIDAK ADA PERUBAHAN) -------------- */
function modifyWorker(type, change) {
    if (gameState.isGameOver) return; 
    
    if (change > 0) {
        if (gameState.freeWorkers <= 0) {
            gameLog("Tidak ada pekerja bebas yang tersisa!", true);
            return;
        }

        if (type === 'hunter') {
            const maxHunters = gameState.buildings.hunterHut.count * gameState.buildings.hunterHut.maxWorkers;
            if (gameState.workers.hunter >= maxHunters) {
                gameLog(`Anda perlu membangun Hunter's Hut (Pemburu) lagi! Kapasitas: ${maxHunters}.`, true);
                return;
            }
        }
        
        gameState.workers[type]++;
        gameState.freeWorkers--;
    } 
    else if (change < 0) {
        if (gameState.workers[type] <= 0) {
            gameLog(`Tidak ada ${type} yang bisa dibebaskan.`);
            return;
        }

        gameState.workers[type]--;
        gameState.freeWorkers++;
    }
    
    // Sinkronisasi visual worker setelah alokasi berubah
    syncVisualWorkers();
    updateUI();
}

function buildBuilding(type) {
    if (gameState.isGameOver) return; 
    const building = gameState.buildings[type];
    const cost = building.cost;

    if (gameState.resources.wood >= cost.wood) {
        gameState.resources.wood -= cost.wood;
        building.count++;
        gameLog(`Berhasil membangun ${type}! Biaya: ${cost.wood} Kayu.`);
    } else {
        gameLog(`Tidak cukup sumber daya untuk membangun ${type}! Butuh ${cost.wood} Kayu.`, true);
    }
    
    updateUI();
}

function handleWanderer(action) {
    if (gameState.isGameOver) return; 
    if (!gameState.events.isWandererPresent) return;

    const foodCost = gameState.events.wandererFoodCost;

    if (action === 'accept') {
        if (gameState.resources.food >= foodCost) {
            gameState.resources.food -= foodCost;
            gameState.population++;
            gameState.freeWorkers++;
            gameState.maxWorkers++;
            gameLog(`👏 Anda menerima Pengembara! Populasi bertambah menjadi ${gameState.population}. Makanan berkurang: ${foodCost}.`);
        } else {
            gameLog(`Makanan tidak cukup (${foodCost}🥩)! Pengembara masih menunggu, tapi akan pergi pada jam 15:00.`, true);
            return; 
        }
    } else if (action === 'reject') {
        gameLog(`Anda menolak Pengembara. Mereka pergi mencari tempat lain.`);
    }

    gameState.events.isWandererPresent = false;
    updateUI();
}

/* -------------- E. UPDATE UI & CANVAS (LOGIKA PERGERAKAN DIPERBARUI) -------------- */
function updateUI() {
    const currentHour = gameState.time.hour.toString().padStart(2, '0');
    const timeStatus = gameState.time.isNight ? '🔴 Malam' : '☀️ Siang';
    timeDisplay.textContent = `${timeStatus} | Hari ${gameState.time.dayCount} (${currentHour}:00)`;
    
    document.getElementById('wood-stat').textContent = `${gameState.resources.wood} / ${gameState.storage.woodMax}`;
    document.getElementById('food-stat').textContent = `${gameState.resources.food} / ${gameState.storage.foodMax}`;
    document.getElementById('population-stat').textContent = gameState.population;
    
    document.getElementById('free-workers-stat-short').textContent = gameState.freeWorkers;
    
    document.getElementById('gatherer-stat-short').textContent = gameState.workers.gatherer;
    document.getElementById('guard-stat-short').textContent = gameState.workers.guard;
    document.getElementById('hunter-stat-short').textContent = gameState.workers.hunter;

    const maxHunters = gameState.buildings.hunterHut.count * gameState.buildings.hunterHut.maxWorkers;
    document.getElementById('max-hunter-stat-short').textContent = maxHunters;

    const hunterHutCost = gameState.buildings.hunterHut.cost.wood;
    document.getElementById('build-hunterhut-btn').disabled = gameState.resources.wood < hunterHutCost;
    document.getElementById('hunterhut-count').textContent = gameState.buildings.hunterHut.count;

    const storageHutCost = gameState.buildings.storageHut.cost.wood;
    document.getElementById('build-storagehut-btn').disabled = gameState.resources.wood < storageHutCost;
    document.getElementById('storagehut-count').textContent = gameState.buildings.storageHut.count;

    const wandererPanel = document.getElementById('wanderer-panel');
    const wandererCostDisplay = document.getElementById('wanderer-cost');
    const wandererAcceptBtn = document.getElementById('wanderer-accept-btn');
    
    if (gameState.events.isWandererPresent) {
        wandererPanel.style.display = 'block';
        wandererCostDisplay.textContent = gameState.events.wandererFoodCost;
        wandererAcceptBtn.disabled = gameState.resources.food < gameState.events.wandererFoodCost;
    } else {
        wandererPanel.style.display = 'none';
    }
}

/* -------------- F. FUNGSI GAMBAR CANVAS -------------- */

function drawBonfire(time) {
    if (!BONFIRE_POS) return; 
    if (!gameState.bonfire.isBurning) {
        ctx.fillStyle = '#334155';
        ctx.beginPath();
        ctx.arc(BONFIRE_POS.x, BONFIRE_POS.y, 8, 0, Math.PI * 2); 
        ctx.fill();
        return; 
    }
    const colorIntensity = Math.sin(time / 150) * 0.1 + 0.9;
    const red = Math.floor(255 * colorIntensity);
    const orange = Math.floor(165 * colorIntensity);
    ctx.fillStyle = time % 400 < 200 ? `rgb(${red}, 68, 68)` : `rgb(${red}, ${orange}, 68)`;
    ctx.beginPath();
    ctx.arc(BONFIRE_POS.x, BONFIRE_POS.y, 10, 0, Math.PI * 2); 
    ctx.fill();
    ctx.fillStyle = 'rgba(255, 100, 0, 0.1)';
    ctx.beginPath();
    ctx.arc(BONFIRE_POS.x, BONFIRE_POS.y, 20 + Math.sin(time / 200) * 5, 0, Math.PI * 2);
    ctx.fill();
}

function drawBuildings() {
    if (!BONFIRE_POS) return; 
    const hutCount = gameState.buildings.hunterHut.count;
    for (let i = 0; i < hutCount; i++) {
        const x = BONFIRE_POS.x + 80 + (i * TILE_SIZE * 1.5);
        const y = BONFIRE_POS.y - 20;
        ctx.fillStyle = '#cc7722';
        ctx.beginPath();
        ctx.moveTo(x, y - 20);
        ctx.lineTo(x + TILE_SIZE / 2, y - 40);
        ctx.lineTo(x + TILE_SIZE, y - 20);
        ctx.closePath();
        ctx.fill();
        ctx.fillStyle = '#8b4513';
        ctx.fillRect(x, y - 20, TILE_SIZE, 30);
    }
    const storageCount = gameState.buildings.storageHut.count;
    for (let i = 0; i < storageCount; i++) {
        const x = BONFIRE_POS.x - 80 - TILE_SIZE - (i * TILE_SIZE * 1.5); 
        const y = BONFIRE_POS.y - 20;
        ctx.fillStyle = '#78716c'; 
        ctx.fillRect(x, y - 20, TILE_SIZE * 1.2, 40);
        ctx.fillStyle = '#57534e';
        ctx.fillRect(x, y - 25, TILE_SIZE * 1.2, 5);
    }
}

// --- FUNGSI BARU UNTUK SINKRONISASI VISUAL PEKERJA ---
function syncVisualWorkers() {
    const requiredCount = gameState.population;
    
    // 1. Sinkronisasi jumlah total pekerja
    if (visualWorkers.length !== requiredCount) {
        while (visualWorkers.length < requiredCount) {
            const randomAngle = Math.random() * Math.PI * 2;
            const randomRadius = Math.random() * REST_RADIUS;
            visualWorkers.push({
                x: BONFIRE_POS.x + Math.cos(randomAngle) * randomRadius,
                y: WORKER_START_Y + Math.sin(randomAngle) * randomRadius,
                targetX: null, 
                targetY: null,
                isMoving: true, 
                isTargetedMoving: false, // Tambahan properti untuk transisi cepat
                type: 'freeWorker' 
            });
        }
        while (visualWorkers.length > requiredCount) {
            visualWorkers.pop(); 
        }
    }

    // 2. Tentukan Tipe dan Warna
    let currentIndex = 0;
    
    // Gatherers
    for (let i = 0; i < gameState.workers.gatherer; i++) {
        if (visualWorkers[currentIndex]) visualWorkers[currentIndex].type = 'gatherer';
        currentIndex++;
    }
    // Hunters
    for (let i = 0; i < gameState.workers.hunter; i++) {
        if (visualWorkers[currentIndex]) visualWorkers[currentIndex].type = 'hunter';
        currentIndex++;
    }
    // Guards
    for (let i = 0; i < gameState.workers.guard; i++) {
        if (visualWorkers[currentIndex]) visualWorkers[currentIndex].type = 'guard';
        currentIndex++;
    }
    // Free Workers
    for (let i = 0; i < gameState.freeWorkers; i++) {
        if (visualWorkers[currentIndex]) visualWorkers[currentIndex].type = 'freeWorker';
        currentIndex++;
    }
}

// --- FUNGSI BARU UNTUK GERAK ACAK (STOCHASTIC MOVEMENT) ---
function moveWorkerStochastic(worker, center, radius, minDistance) {
    const SPEED = 0.5; // Kecepatan gerakan
    const STOP_CHANCE = 0.003; // Peluang untuk berhenti/mulai bergerak

    if (Math.random() < STOP_CHANCE) {
        worker.isMoving = !worker.isMoving;
        if (!worker.isMoving) {
            worker.targetX = null;
            worker.targetY = null;
            return; 
        }
    }
    
    if (!worker.isMoving) return; 

    const distToTarget = worker.targetX !== null ? Math.sqrt(Math.pow(worker.x - worker.targetX, 2) + Math.pow(worker.y - worker.targetY, 2)) : 0;
    
    if (worker.targetX === null || distToTarget < 2) {
        // Hasilkan posisi target acak dalam radius
        const angle = Math.random() * Math.PI * 2;
        const dist = Math.random() * radius;
        
        worker.targetX = center.x + Math.cos(angle) * dist;
        worker.targetY = center.y + Math.sin(angle) * dist;
    }

    // Bergerak menuju target
    const dx = worker.targetX - worker.x;
    const dy = worker.targetY - worker.y;
    const dist = Math.sqrt(dx * dx + dy * dy);

    if (dist > 0) {
        worker.x += (dx / dist) * SPEED;
        worker.y += (dy / dist) * SPEED;
    }

    // Boundary Check (Hindari Pusat Api Unggun)
    const currentDistFromBonfire = Math.sqrt(Math.pow(worker.x - BONFIRE_POS.x, 2) + Math.pow(worker.y - BONFIRE_POS.y, 2));
    if (currentDistFromBonfire < minDistance) {
        const angle = Math.atan2(worker.y - BONFIRE_POS.y, worker.x - BONFIRE_POS.x);
        worker.x = BONFIRE_POS.x + Math.cos(angle) * minDistance;
        worker.y = BONFIRE_POS.y + Math.sin(angle) * minDistance;
        worker.targetX = null;
    }
}

// --- FUNGSI BARU UNTUK GERAK CEPAT MENUJU TARGET (TRANSISI) ---
function moveWorkerToTarget(worker, speed) {
    const dx = worker.targetX - worker.x;
    const dy = worker.targetY - worker.y;
    const dist = Math.sqrt(dx * dx + dy * dy);

    if (dist > speed) {
        // Pindahkan
        worker.x += (dx / dist) * speed;
        worker.y += (dy / dist) * speed;
    } else {
        // Target tercapai
        worker.x = worker.targetX;
        worker.y = worker.targetY;
        worker.isTargetedMoving = false; // Hentikan transisi
        worker.targetX = null; 
        worker.targetY = null;
    }
}


function drawWorkers(time) {
    if (!BONFIRE_POS) return; 
    
    syncVisualWorkers(); 

    function drawDot(x, y, color) {
        ctx.fillStyle = color;
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, Math.PI * 2);
        ctx.fill();
    }
    
    const REST_CENTER = { x: BONFIRE_POS.x, y: WORKER_START_Y };
    const WORK_SPOT = { x: 50, y: BONFIRE_POS.y };
    const GUARD_RADIUS = 50;

    visualWorkers.forEach((worker, i) => {
        let x = worker.x;
        let y = worker.y;
        let color = WORKER_COLORS[worker.type] || '#fff'; 
        
        const isProductionWorker = worker.type === 'gatherer' || worker.type === 'hunter';
        
        if (worker.isTargetedMoving) {
             // 1. Fase Lari (Hanya untuk pekerja produksi yang sedang transisi)
            moveWorkerToTarget(worker, TRANSITION_SPEED);
            x = worker.x;
            y = worker.y;
            
        } else if (worker.type === 'guard') {
            // 2. Penjaga (Guard) - Patroli Rotasi
            const angle = i * (Math.PI * 2 / Math.max(1, gameState.workers.guard)) + time / 300;
            x = BONFIRE_POS.x + Math.cos(angle) * GUARD_RADIUS;
            y = BONFIRE_POS.y + Math.sin(angle) * GUARD_RADIUS;
            worker.x = x;
            worker.y = y;

        } else if ((isProductionWorker && gameState.time.isNight) || worker.type === 'freeWorker') {
            // 3. Fase Istirahat (Gerak Acak)
            moveWorkerStochastic(worker, REST_CENTER, REST_RADIUS, MIN_DISTANCE_FROM_BONFIRE);
            x = worker.x;
            y = worker.y;
            
        } else if (isProductionWorker && !gameState.time.isNight) {
            // 4. Fase Kerja (Animasi Sine/Cosine)
            const offset = worker.type === 'gatherer' ? 0 : 30; 
            const xOffset = Math.sin(time / (200 + i * 50)) * 20;
            const yOffset = Math.cos(time / (150 + i * 40)) * 10;
            x = WORK_SPOT.x + xOffset;
            y = WORK_SPOT.y + offset + (i * 15) + yOffset;
            
            worker.x = x;
            worker.y = y;
        }

        drawDot(x, y, color);
    });
    
    // Penyerang (Raider)
    if (gameState.time.isAttackPhase && gameState.defense.raidStrength > 0) {
        const raiderCount = gameState.defense.raidStrength; 
        for (let i = 0; i < raiderCount; i++) {
             const x = canvas.width - 50 + Math.sin(time / (100 + i * 30)) * 20; 
             const y = WORKER_START_Y + (i * 10) + Math.cos(time / 120) * 5; 
             drawDot(x, y, WORKER_COLORS['raider']);
        }
    }
}

const DAY_COLOR = '#65a30d'; 
const NIGHT_COLOR = '#0f172a'; 

function drawVillage(time) {
    if (gameState.isGameOver) return; 
    let currentColor = DAY_COLOR;
    const hour = gameState.time.hour;
    let transitionFactor = 0;
    
    if (hour >= 18 && hour < 20) {
        transitionFactor = (hour - 18) / 2;
        currentColor = interpolateColor(DAY_COLOR, NIGHT_COLOR, transitionFactor);
    } 
    else if (hour >= 6 && hour < 8) {
        transitionFactor = (hour - 6) / 2;
        currentColor = interpolateColor(NIGHT_COLOR, DAY_COLOR, transitionFactor);
    }
    else if (hour >= 20 || hour < 6) {
        currentColor = NIGHT_COLOR;
    }
    else {
        currentColor = DAY_COLOR;
    }

    ctx.fillStyle = currentColor;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    drawBuildings(); 
    drawBonfire(time); 
    drawWorkers(time); 

    animationFrameId = requestAnimationFrame(drawVillage);
}


function init() {
    canvas = document.getElementById('gameCanvas');
    
    if (canvas) {
        ctx = canvas.getContext('2d');
        
        BONFIRE_POS = { x: canvas.width / 2, y: canvas.height / 2 + 50 }; 
        WORKER_START_Y = BONFIRE_POS.y + 40;
        
        if (!gameState.isGameOver) {
            animationFrameId = requestAnimationFrame(drawVillage);
        }
    }
    
    if (!gameState.isGameOver) {
        timeIntervalId = setInterval(updateTime, 1000); 
        cityIntervalId = setInterval(updateCity, 1000); 
    }
    
    updateUI(); 
    syncVisualWorkers(); // Pastikan visualWorkers terinisialisasi
}

document.addEventListener('DOMContentLoaded', init);

    </script>
</body>
</html>
