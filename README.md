<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🎰 Rifas AI - 100% Automatizado</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://unpkg.com/particles.js@2.0.0/particles.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap');
        * { font-family: 'Poppins', sans-serif; }
        .glass { 
            background: rgba(255,255,255,0.1); 
            backdrop-filter: blur(20px); 
            border: 1px solid rgba(255,255,255,0.2);
        }
        .progress-ring { transform: rotate(-90deg); }
        .progress-ring-circle { 
            stroke-dasharray: 251.2; 
            stroke-dashoffset: 251.2; 
            transition: stroke-dashoffset 1s ease-in-out;
        }
        .neon-glow { box-shadow: 0 0 20px rgba(59,130,246,0.5); }
    </style>
</head>
<body class="bg-gradient-to-br from-purple-900 via-blue-900 to-indigo-900 min-h-screen text-white overflow-x-hidden">

<!-- 🔒 IndexedDB para datos persistentes -->
<script>
class Database {
    constructor() {
        this.db = null;
        this.init();
    }
    
    init() {
        const request = indexedDB.open('RifasAI', 1);
        request.onupgradeneeded = (e) => {
            const db = e.target.result;
            db.createObjectStore('raffles', { keyPath: '_id', autoIncrement: true });
            db.createObjectStore('tickets', { keyPath: '_id', autoIncrement: true });
            db.createObjectStore('sales', { keyPath: '_id', autoIncrement: true });
        };
        request.onsuccess = (e) => this.db = e.target.result;
    }
    
    async save(table, data) {
        return new Promise((resolve) => {
            const tx = this.db.transaction(table, 'readwrite');
            const store = tx.objectStore(table);
            const request = store.add(data);
            request.onsuccess = () => resolve(request.result);
        });
    }
    
    async getAll(table) {
        return new Promise((resolve) => {
            const tx = this.db.transaction(table, 'readonly');
            const store = tx.objectStore(table);
            const request = store.getAll();
            request.onsuccess = () => resolve(request.result);
        });
    }
    
    async update(table, id, data) {
        return new Promise((resolve) => {
            const tx = this.db.transaction(table, 'readwrite');
            const store = tx.objectStore(table);
            store.get(id).onsuccess = (e) => {
                Object.assign(e.target.result, data);
                store.put(e.target.result);
                resolve();
            };
        });
    }
}

const db = new Database();
</script>

<!-- HEADER -->
<header class="glass sticky top-0 z-50 px-6 py-4 backdrop-blur-xl">
    <div class="max-w-7xl mx-auto flex justify-between items-center">
        <div class="flex items-center space-x-3">
            <div class="w-14 h-14 bg-gradient-to-r from-yellow-400 to-orange-500 rounded-2xl flex items-center justify-center shadow-2xl neon-glow">
                <i class="fas fa-ticket-alt text-2xl"></i>
            </div>
            <div>
                <h1 class="text-3xl font-bold bg-gradient-to-r from-white to-gray-200 bg-clip-text text-transparent">Rifas AI</h1>
                <p class="text-xs opacity-75">🤖 100% Automatizado con IA</p>
            </div>
        </div>
        <div class="flex items-center space-x-4">
            <button id="adminBtn" class="px-6 py-3 bg-white/20 hover:bg-white/30 rounded-2xl transition-all duration-300 flex items-center space-x-2">
                <i class="fas fa-cog"></i><span>Admin</span>
            </button>
            <button id="cartBtn" class="relative px-6 py-3 bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-600 hover:to-teal-700 rounded-2xl transition-all duration-300 flex items-center space-x-2 shadow-xl hover:shadow-2xl">
                <i class="fas fa-shopping-cart"></i>
                <span>Carrito</span>
                <span id="cartCount" class="bg-red-500 text-xs w-6 h-6 rounded-full flex items-center justify-center -mr-2 -mt-1">0</span>
            </button>
        </div>
    </div>
</header>

<!-- HERO + PROGRESO GLOBAL -->
<section class="max-w-7xl mx-auto px-6 py-24">
    <div class="text-center mb-24">
        <div class="inline-block animate-pulse mb-8">
            <i class="fas fa-rocket text-6xl text-yellow-400"></i>
        </div>
        <h2 class="text-6xl md:text-8xl font-black bg-gradient-to-r from-yellow-400 via-orange-400 to-red-500 bg-clip-text text-transparent mb-8 leading-tight">
            ¡GANÁ PREMIOS<br>INCREÍBLES!
        </h2>
        <p class="text-2xl opacity-90 max-w-4xl mx-auto leading-relaxed">
            Boletos digitales • Sorteos en vivo • Pagos seguros • 
            <span class="font-black text-yellow-400 animate-pulse">100% transparente</span>
        </p>
    </div>

    <!-- BARRA PROGRESO GLOBAL AVANZADA -->
    <div class="glass p-12 rounded-3xl mb-20 max-w-6xl mx-auto shadow-2xl hover:shadow-3xl transition-all duration-500">
        <div class="flex flex-col lg:flex-row items-center justify-center lg:justify-between gap-12">
            <!-- Círculo Progreso 3D -->
            <div class="relative group">
                <svg class="w-40 h-40 lg:w-52 lg:h-52 mx-auto progress-ring" viewBox="0 0 100 100">
                    <defs>
                        <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="100%">
                            <stop offset="0%" stop-color="#10b981"/>
                            <stop offset="50%" stop-color="#34d399"/>
                            <stop offset="100%" stop-color="#6ee7b7"/>
                        </linearGradient>
                    </defs>
                    <circle cx="50" cy="50" r="42" stroke="#374151" stroke-width="6" fill="transparent" stroke-linecap="round"/>
                    <circle id="globalRing" class="progress-ring-circle" cx="50" cy="50" r="42" stroke="url(#grad1)" stroke-width="6" fill="transparent" stroke-linecap="round"/>
                </svg>
                <div class="absolute inset-0 flex flex-col items-center justify-center">
                    <div id="globalPercent" class="text-4xl lg:text-5xl font-black text-transparent bg-gradient-to-r from-emerald-400 to-teal-400 bg-clip-text drop-shadow-2xl">0%</div>
                    <div class="text-lg opacity-90 font-semibold mt-1">PROGRESO GLOBAL</div>
                </div>
                <!-- Partículas -->
                <div id="particles" class="absolute inset-0 w-full h-full"></div>
            </div>
            
            <!-- Stats Detallados -->
            <div class="flex-1 lg:max-w-md">
                <div class="grid grid-cols-2 gap-6 mb-8">
                    <div class="text-center p-6 bg-white/10 backdrop-blur-sm rounded-2xl hover:bg-white/20 transition-all duration-300">
                        <div class="text-3xl font-black text-emerald-400 mb-2" id="totalRaised">$0</div>
                        <div class="text-sm opacity-75 uppercase tracking-wider">Recaudado</div>
                    </div>
                    <div class="text-center p-6 bg-white/10 backdrop-blur-sm rounded-2xl hover:bg-white/20 transition-all duration-300">
                        <div class="text-3xl font-black text-blue-400 mb-2" id="totalSold">0</div>
                        <div class="text-sm opacity-75 uppercase tracking-wider">Boletos Vendidos</div>
                    </div>
                </div>
                <div class="w-full bg-white/10 rounded-2xl p-4">
                    <div class="flex items-center justify-between mb-3">
                        <span class="font-semibold">Progreso:</span>
                        <span id="progressText">0/1000</span>
                    </div>
                    <div class="w-full bg-white/20 rounded-xl h-4 overflow-hidden">
                        <div id="globalBar" class="h-full bg-gradient-to-r from-emerald-400 via-teal-400 to-emerald-500 rounded-xl shadow-inner transition-all duration-1000 ease-out" style="width:0%"></div>
                    </div>
                </div>
                <div class="flex items-center justify-between mt-4 text-sm opacity-75">
                    <span>⏰ Sorteo en:</span>
                    <span id="countdown">07d 12:34:56</span>
                </div>
            </div>
        </div>
    </div>

    <!-- RIFAS DISPONIBLES -->
    <div class="mb-24">
        <h3 class="text-4xl font-bold text-center mb-16 bg-gradient-to-r from-white to-gray-300 bg-clip-text text-transparent">
            🎁 RIFAS DISPONIBLES
        </h3>
        <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8" id="rafflesGrid">
            <!-- Rifas cargadas dinámicamente -->
        </div>
    </div>
</section>

<!-- CARRITO MODAL -->
<div id="cartModal" class="fixed inset-0 bg-black/70 backdrop-blur-md z-[9999] hidden items-center justify-center p-6 animate-fadeIn">
    <div class="glass max-w-2xl w-full max-h-[90vh] overflow-y-auto rounded-3xl p-10 shadow-2xl scale-95 animate-scaleIn">
        <div class="flex justify-between items-center mb-10">
            <h3 class="text-4xl font-black bg-gradient-to-r from-emerald-400 to-teal-500 bg-clip-text text-transparent">
                🛒 Mi Carrito
            </h3>
            <button onclick="hideCart()" class="text-3xl hover:scale-110 transition-transform duration-200">&times;</button>
        </div>
        <div id="cartContent" class="space-y-6 mb-10"></div>
        <div class="border-t border-white/20 pt-10">
            <div class="flex justify-between text-3xl font-black mb-8">
                <span>Total:</span>
                <span id="finalTotal" class="text-emerald-400">$0.00</span>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <button onclick="payWith('stripe')" class="bg-gradient-to-r from-purple-500 to-indigo-600 hover:from-purple-600 hover:to-indigo-700 p-5 rounded-2xl font-bold text-xl shadow-xl hover:shadow-2xl transition-all duration-300 flex items-center justify-center space-x-3">
                    <i class="fab fa-cc-stripe text-2xl"></i>
                    <span>Stripe</span>
                </button>
                <button onclick="payWith('paypal')" class="bg-gradient-to-r from-blue-500 to-blue-600 hover:from-blue-600 hover:to-blue-700 p-5 rounded-2xl font-bold text-xl shadow-xl hover:shadow-2xl transition-all duration-300 flex items-center justify-center space-x-3">
                    <i class="fab fa-paypal text-2xl"></i>
                    <span>PayPal</span>
                </button>
                <button onclick="payWith('mercadopago')" class="bg-gradient-to-r from-orange-500 to-orange-600 hover:from-orange-600 hover:to-orange-700 p-5 rounded-2xl font-bold text-xl shadow-xl hover:shadow-2xl transition-all duration-300 flex items-center justify-center space-x-3">
                    <i class="fas fa-wallet text-2xl"></i>
                    <span>Mercado Pago</span>
                </button>
            </div>
        </div>
    </div>
</div>

<!-- ADMIN PANEL -->
<div id="adminPanel" class="fixed inset-0 bg-black/90 backdrop-blur-md z-[9999] hidden flex items-center justify-center p-6">
    <div class="glass max-w-4xl w-full max-h-[95vh] overflow-y-auto rounded-3xl p-12">
        <div class="flex justify-between items-center mb-12">
            <h2 class="text-5xl font-black bg-gradient-to-r from-purple-400 to-pink-400 bg-clip-text text-transparent">
                ⚙️ Panel de Administración
            </h2>
            <button onclick="toggleAdmin()" class="text-3xl hover:scale-110">&times;</button>
        </div>
        
        <!-- Crear Rifa -->
        <div class="bg-white/10 p-8 rounded-3xl mb-12">
            <h3 class="text-3xl font-bold mb-8">➕ Nueva Rifa</h3>
            <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
                <input id="raffleTitle" placeholder="Título de la rifa" class="p-4 rounded-2xl bg-white/20 border border-white/30 focus:border-blue-400 focus:outline-none text-lg w-full">
                <input id="rafflePrice" type="number" placeholder="$ Precio boleto" class="p-4 rounded-2xl bg-white/20 border border-white/30 focus:border-blue-400 focus:outline-none text-lg w-full">
                <input id="raffleTickets" type="number" placeholder="Total boletos" class="p-4 rounded-2xl bg-white/20 border border-white/30 focus:border-blue-400 focus:outline-none text-lg w-full">
                <input id="rafflePrize" type="number" placeholder="$ Valor premio" class="p-4 rounded-2xl bg-white/20 border border-white/30 focus:border-blue-400 focus:outline-none text-lg w-full md:col-span-2">
                <textarea id="raffleDesc" placeholder="Descripción atractiva..." class="p-4 rounded-2xl bg-white/20 border border-white/30 focus:border-blue-400 focus:outline-none text-lg w-full md:col-span-2 resize-none h-24"></textarea>
                <button onclick="createRaffle()" class="md:col-span-3 bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-600 hover:to-teal-700 p-6 rounded-3xl font-black text-2xl shadow-2xl hover:shadow-3xl transition-all duration-300">
                    🚀 Crear Rifa Ahora
                </button>
            </div>
        </div>
        
        <!-- Estadísticas -->
        <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 mb-12">
            <div class="glass p-8 rounded-3xl text-center hover:scale-105 transition-all">
                <div class="text-4xl font-black text-emerald-400 mb-2" id="adminRevenue">$0</div>
                <div class="opacity-75">Ingresos Totales</div>
            </div>
            <div class="glass p-8 rounded-3xl text-center hover:scale-105 transition-all">
                <div class="text-4xl font-black text-blue-400 mb-2" id="adminTickets">0</div>
                <div class="opacity-75">Boletos Vendidos</div>
            </div>
            <div class="glass p-8 rounded-3xl text-center hover:scale-105 transition-all">
                <div class="text-4xl font-black text-purple-400 mb-2" id="adminRaffles">0</div>
                <div class="opacity-75">Rifas Activas</div>
            </div>
            <div class="glass p-8 rounded-3xl text-center hover:scale-105 transition-all">
                <div class="text-4xl font-black text-yellow-400 mb-2" id="adminConversion">0%</div>
                <div class="opacity-75">Conversión</div>
            </div>
        </div>
        
        <!-- Lista Rifas -->
        <div class="space-y-4">
            <h4 class="text-2xl font-bold mb-6">📋 Rifas Creadas</h4>
            <div id="adminRafflesList"></div>
        </div>
    </div>
</div>

<!-- ANIMACIONES CSS -->
<style>
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes scaleIn { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
.animate-fadeIn { animation: fadeIn 0.3s ease-out; }
.animate-scaleIn { animation: scaleIn 0.3s ease-out; }
.line-clamp-2 { display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
.line-clamp-3 { display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; overflow: hidden; }
</style>

<script>
class RifasAI {
    constructor() {
        this.raffles = [];
        this.cart = [];
        this.sales = [];
        this.socket = io('wss://rifas-ai-backend.onrender.com'); // Backend público demo
        this.init();
    }
    
    async init() {
        await this.loadData();
        this.renderRaffles();
        this.updateGlobalStats();
        this.startCountdown();
        this.setupEventListeners();
        this.startParticles();
        this.updateLoop();
    }
    
    async loadData() {
        this.raffles = await db.getAll('raffles');
        this.cart = JSON.parse(localStorage.getItem('rifasCart') || '[]');
        this.sales = await db.getAll('sales');
        
        // Datos demo si no hay rifas
        if (this.raffles.length === 0) {
            this.raffles = [
                {
                    _id: '1', title: '🏎️ Auto Tesla Model 3', description: 'Último modelo 2024, 0km, full equipado', 
                    image: 'https://images.unsplash.com/photo-1552519507-da3b142c6e3d?w=500', 
                    ticketPrice: 25, totalTickets: 2000, soldTickets: 1423, prizeValue: 45000, progress: 71
                },
                {
                    _id: '2', title: '💰 $25,000 EN EFECTIVO', description: 'Dinero fresco para tus proyectos', 
                    image: 'https://images.unsplash.com/photo-1579621970563-ebec7560ff3e?w=500', 
                    ticketPrice: 10, totalTickets: 5000, soldTickets: 3876, prizeValue: 25000, progress: 77
                },
                {
                    _id: '3', title: '✈️ Viaje a Dubai 7 días', description: 'Vuelos + hotel 5* + $2000 gastos', 
                    image: 'https://images.unsplash.com/photo-1522071820081-009f0129c71c?w=500', 
                    ticketPrice: 50, totalTickets: 1000, soldTickets: 623, prizeValue: 8000, progress: 62
                }
            ];
            this.raffles.forEach(r => db.save('raffles', r));
        }
    }
    
    renderRaffles() {
        document.getElementById('rafflesGrid').innerHTML = this.raffles.map(r => `
            <div class="glass p-8 rounded-3xl hover:scale-[1.02] transition-all duration-500 cursor-pointer group relative overflow-hidden" onclick="app.buyTickets('${r._id}')">
                <div class="absolute top-6 right-6 bg-gradient-to-br from-red-500 to-pink-500 text-white px-4 py-2 rounded-2xl text-lg font-bold shadow-2xl">
                    ${r.progress}% Vendido
                </div>
                <img src="${r.image}" alt="${r.title}" class="w-full h-52 object-cover rounded-2xl mb-6 group-hover:scale-110 transition-transform duration-500 shadow-2xl">
                <h3 class="text-2xl font-bold mb-4 line-clamp-2">${r.title}</h3>
                <p class="opacity-75 mb-8 line-clamp-3">${r.description}</p>
                
                <div class="grid grid-cols-3 gap-4 mb-8">
                    <div class="text-center p-4 bg-white/10 rounded-2xl">
                        <div class="text-2xl font-bold text-emerald-400">$${r.ticketPrice}</div>
                        <div class="text-xs opacity-75">Por boleto</div>
                    </div>
                    <div class="text-center p-4 bg-white/10 rounded-2xl">
                        <div class="text-2xl font-bold">${r.soldTickets}/${r.totalTickets}</div>
                        <div class="text-xs opacity-75">Vendidos</div>
                    </div>
                    <div class="text-center p-4 bg-white/10 rounded-2xl">
                        <div class="text-2xl font-bold text-yellow-400">$${r.prizeValue.toLocaleString()}</div>
                        <div class="text-xs opacity-75">Premio</div>
                    </div>
                </div>
                
                <div class="w-full bg-gradient-to-r from-blue-500 to-purple-600 p-1 rounded-2xl group-hover:from-blue-600 group-hover:to-purple-700 mb-6">
                    <button class="w-full bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 p-5 rounded-2xl font-bold text-xl transition-all duration-300 flex items-center justify-center space-x-3 shadow-xl hover:shadow-2xl">
                        <i class="fas fa-ticket-alt"></i>
                        <span>COMPRAR BOLETOS</span>
                    </button>
                </div>
                
                <div class="w-full bg-white/10 rounded-2xl p-3">
                    <div class="w-full bg-white/20 rounded-xl h-3 overflow-hidden">
                        <div class="h-full bg-gradient-to-r from-emerald-400 to-teal-500 rounded-xl shadow-inner transition-all duration-1000" style="width:${r.progress}%"></div>
                    </div>
                </div>
            </div>
        `).join('');
    }
    
    updateGlobalStats() {
        const totalSold = this.raffles.reduce((sum, r) => sum + r.soldTickets, 0);
        const totalTickets = this.raffles.reduce((sum, r) => sum + r.totalTickets, 0);
        const totalRevenue = this.raffles.reduce((sum, r) => sum + (r.soldTickets * r.ticketPrice), 0);
        const progress = Math.round((totalSold / totalTickets) * 100);
        
        document.getElementById('globalPercent').textContent = `${progress}%`;
        document.getElementById('globalRing').style.strokeDashoffset = 251.2 - (251.2 * progress / 100);
        document.getElementById('globalBar').style.width = `${progress}%`;
        document.getElementById('totalRaised').textContent = `$${totalRevenue.toLocaleString()}`;
        document.getElementById('totalSold').textContent = totalSold.toLocaleString();
        document.getElementById('progressText').textContent = `${totalSold}/${totalTickets}`;
        
        // Actualizar admin stats
        document.getElementById('adminRevenue').textContent = `$${totalRevenue.toLocaleString()}`;
        document.getElementById('adminTickets').textContent = totalSold.toLocaleString();
        document.getElementById('adminRaffles').textContent = this.raffles.filter(r => r.progress < 100).length;
    }
    
    startCountdown() {
        const endDate = Date.now() + 7 * 24 * 60 * 60 * 1000; // 7 días
        setInterval(() => {
            const diff = endDate - Date.now();
            const days = Math.floor(diff / (1000*60*60*24));
            const hours = Math.floor((diff%(1000*60*60*24))/(1000*60*60));
            const minutes = Math.floor((diff%(1000*60*60))/(1000*60));
            const seconds = Math.floor((diff%1000)/1000);
            document.getElementById('countdown').textContent = 
                `${days}d ${hours.toString().padStart(2,'0')}h ${minutes.toString().padStart(2,'0')}m ${seconds.toString().padStart(2,'0')}s`;
        }, 1000);
    }
    
    setupEventListeners() {
        document.getElementById('adminBtn').onclick = () => toggleAdmin();
        document.getElementById('cartBtn').onclick = () => showCart();
        
        // Actualizar carrito cada segundo
        setInterval(() => this.updateCartUI(), 1000);
    }
    
    startParticles() {
        particlesJS('particles', {
            particles: {
                number: { value: 30 },
                color: { value: '#10b981' },
                shape: { type: 'circle' },
                opacity: { value: 0.5, random: true },
                size: { value: 4, random: true },
                move: { speed: 1 }
            },
            interactivity: {
                events: { onhover: { enable: true, mode: 'repulse' } }
            }
        });
    }
    
    buyTickets(raffleId) {
        const raffle = this.raffles.find(r => r._id === raffleId);
        const quantity = parseInt(prompt(`¿Cuántos boletos de $${raffle.ticketPrice} quieres?\n(1-${raffle.totalTickets - raffle.soldTickets})`) || 0);
        
        if (quantity > 0 && quantity <= raffle.totalTickets - raffle.soldTickets) {
            const cartItem = this.cart.find(item => item.raffleId === raffleId);
            if (cartItem) {
                cartItem.quantity += quantity;
            } else {
                this.cart.push({ raffleId, quantity, raffle });
            }
            localStorage.setItem('rifasCart', JSON.stringify(this.cart));
            this.updateCartUI();
            this.showNotification(`✅ ${quantity} boleto(s) agregado al carrito!`);
        }
    }
    
    updateCartUI() {
        const totalItems = this.cart.reduce((sum, item) => sum + item.quantity, 0);
        document.getElementById('cartCount').textContent = totalItems;
        
        if (totalItems > 0) {
            document.getElementById('cartBtn').classList.add('animate-pulse');
        } else {
            document.getElementById('cartBtn').classList.remove('animate-pulse');
        }
    }
    
    updateLoop() {
        // Simular ventas en tiempo real
        if (Math.random() < 0.02) {
            const raffle = this.raffles[Math.floor(Math.random() * this.raffles.length)];
            if (raffle.soldTickets < raffle.totalTickets) {
                raffle.soldTickets++;
                raffle.progress = Math.round((raffle.soldTickets / raffle.totalTickets) * 100);
                db.update('raffles', raffle._id, raffle);
                this.updateGlobalStats();
                this.renderRaffles();
            }
        }
        requestAnimationFrame(() => this.updateLoop());
    }
    
    showNotification(message) {
        const notification = document.createElement('div');
        notification.className = 'fixed top-24 right-6 glass p-6 rounded-3xl shadow-2xl z-[10000] animate-slideInRight max-w-sm';
        notification.innerHTML = `<i class="fas fa-check-circle text-3xl text-emerald-400 mb-3"></i><p class="font-bold text-lg">${message}</p>`;
        document.body.appendChild(notification);
        setTimeout(() => notification.remove(), 4000);
    }
}

// FUNCIONES GLOBALES
const app = new RifasAI();

function toggleAdmin() {
    const panel = document.getElementById('adminPanel');
    panel.classList.toggle('hidden');
}

function showCart() {
    document.getElementById('cartModal').classList.remove('hidden');
    renderCart();
}

function hideCart() {
    document.getElementById('cartModal').classList.add('hidden');
}

function renderCart() {
    const cartContent = document.getElementById('cartContent');
    const finalTotal = app.cart.reduce((sum, item) => sum + (item.quantity * item.raffle.ticketPrice), 0);
    
    cartContent.innerHTML = app.cart.map(item => `
        <div class="flex items-center justify-between p-6 bg-white/10 rounded-2xl hover:bg-white/20 transition-all">
            <div class="flex items-center space-x-4">
                <img src="${item.raffle.image}" class="w-20 h-20 rounded-xl object-cover">
                <div>
                    <h4 class="font-bold text-xl">${item.raffle.title}</h4>
                    <p class="opacity-75">$${item.raffle.ticketPrice} x ${item.quantity}</p>
                </div>
            </div>
            <div class="text-right">
                <div class="text-2xl font-bold text-emerald-400">
                    $${(item.raffle.ticketPrice * item.quantity).toLocaleString()}
                </div>
                <button onclick="removeFromCart('${item.raffleId}')" class="mt-2 text-red-400 hover:text-red-500 text-sm font-bold">
                    <i class="fas fa-trash"></i> Quitar
                </button>
            </div>
        </div>
    `).join('') || '<div class="text-center py-20 opacity-50"><i class="fas fa-shopping-cart text-6xl mb-4"></i><p class="text-2xl">Carrito vacío</p></div>';
    
    document.getElementById('finalTotal').textContent = `$${finalTotal.toLocaleString()}`;
}

function removeFromCart(raffleId) {
    app.cart = app.cart.filter(item => item.raffleId !== raffleId);
    localStorage.setItem('rifasCart', JSON.stringify(app.cart));
    app.updateCartUI();
    renderCart();
}

function payWith(method) {
    const total = app.cart.reduce((sum, item) => sum + (item.quantity * item.raffle.ticketPrice), 0);
    
    if (confirm(`🧾 Confirmar pago de $${total.toLocaleString()} con ${method.toUpperCase()}?`)) {
        // Simular pago exitoso
        app.showNotification(`✅ ¡Pago exitoso con ${method}! Boletos enviados por email`);
        
        // Registrar venta
        app.cart.forEach(item => {
            for(let i = 0; i < item.quantity; i++) {
                db.save('sales', {
                    raffleId: item.raffleId,
                    amount: item.raffle.ticketPrice,
                    method,
                    date: new Date().toISOString()
                });
            }
        });
        
        // Limpiar carrito
        app.cart = [];
        localStorage.removeItem('rifasCart');
        hideCart();
        app.updateCartUI();
    }
}

async function createRaffle() {
    const raffle = {
        _id: Date.now().toString(),
        title: document.getElementById('raffleTitle').value || 'Nueva Rifa',
        description: document.getElementById('raffleDesc').value || '¡Participa ya!',
        ticketPrice: parseFloat(document.getElementById('rafflePrice').value) || 10,
        totalTickets: parseInt(document.getElementById('raffleTickets').value) || 1000,
        prizeValue: parseFloat(document.getElementById('rafflePrize').value) || 10000,
        soldTickets: 0,
        progress: 0,
        image: 'https://images.unsplash.com/photo-1552519507-da3b142c6e3d?w=500'
    };
    
    await db.save('raffles', raffle);
    app.raffles.push(raffle);
    app.renderRaffles();
    app.updateGlobalStats();
    app.showNotification('🎉 Rifa creada exitosamente!');
    
    // Limpiar formulario
    document.querySelectorAll('#adminPanel input, #adminPanel textarea').forEach(el => el.value = '');
}

// Inicialización final
document.addEventListener('DOMContentLoaded', () => {
    document.body.classList.add('animate-fadeIn');
});
</script>

<!-- PWA Manifest -->
<link rel="manifest" href="data:application/manifest+json,{
    'name': 'Rifas AI',
    'short_name': 'RifasAI',
    'start_url': '/',
    'display': 'standalone',
    'background_color': '#1e293b',
    'theme_color': '#3b82f6',
    'icons': [{'src': 'data:image/svg+xml;base64,...', 'sizes': '192x192', 'type': 'image/png'}]
}">
<meta name="theme-color" content="#3b82f6">

<!-- Service Worker -->
<script>
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('data:text/javascript;base64,...');
}
</script>

</body>
</html>
