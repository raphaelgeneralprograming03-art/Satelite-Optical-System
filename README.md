<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SISTEMA SROS - Sincronização por Rede Óptica Satelital</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Inter:wght@300;400;600;700&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            background-color: #030712;
            color: #e5e7eb;
            overflow: hidden;
            user-select: none;
        }

        .font-mono-tech {
            font-family: 'Share Tech Mono', monospace;
        }

        /* Glassmorphism Styles */
        .hud-panel {
            background: rgba(10, 15, 30, 0.75);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(0, 243, 255, 0.2);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.5), inset 0 0 12px rgba(0, 243, 255, 0.05);
        }

        .hud-panel-active {
            border-color: rgba(0, 243, 255, 0.6);
            box-shadow: 0 0 15px rgba(0, 243, 255, 0.3);
        }

        .hud-border-corner {
            position: relative;
        }
        .hud-border-corner::before {
            content: '';
            position: absolute;
            top: -1px; left: -1px; width: 8px; height: 8px;
            border-top: 2px solid #00f3ff;
            border-left: 2px solid #00f3ff;
        }
        .hud-border-corner::after {
            content: '';
            position: absolute;
            bottom: -1px; right: -1px; width: 8px; height: 8px;
            border-bottom: 2px solid #00f3ff;
            border-right: 2px solid #00f3ff;
        }

        /* Custom Scrollbars */
        ::-webkit-scrollbar {
            width: 4px;
            height: 4px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(5, 10, 20, 0.5);
        }
        ::-webkit-scrollbar-thumb {
            background: #00f3ff55;
            border-radius: 2px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #00f3ff;
        }

        /* Pulse animations */
        @keyframes pulse-cyan {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.4; transform: scale(0.98); }
        }
        .animate-pulse-cyan {
            animation: pulse-cyan 2s infinite ease-in-out;
        }

        /* Glowing text */
        .glow-cyan { text-shadow: 0 0 8px rgba(0, 243, 255, 0.6); }
        .glow-green { text-shadow: 0 0 8px rgba(0, 255, 136, 0.6); }
        .glow-red { text-shadow: 0 0 8px rgba(255, 50, 50, 0.6); }
    </style>
</head>
<body class="h-screen w-screen flex flex-col justify-between p-2 sm:p-4 text-xs sm:text-sm">

    <!-- Top Navigation & Header Panel -->
    <header class="hud-panel hud-border-corner p-3 rounded-lg flex flex-wrap justify-between items-center z-20 gap-2">
        <div class="flex items-center gap-3">
            <div class="w-8 h-8 rounded bg-cyan-500/20 border border-cyan-400 flex items-center justify-center text-cyan-400">
                <i data-lucide="satellite" class="w-5 h-5"></i>
            </div>
            <div>
                <h1 class="font-mono-tech text-base sm:text-lg font-bold text-cyan-400 tracking-wider flex items-center gap-2">
                    SISTEMA SROS <span class="text-xs px-2 py-0.5 rounded bg-cyan-950 border border-cyan-500/40 text-cyan-300">NBR 8.0 / ISO 128</span>
                </h1>
                <p class="text-gray-400 text-[10px] sm:text-xs">SINCRONIZAÇÃO POR REDE ÓPTICA SATELITAL - MONITORAMENTO DE ENGENHARIA</p>
            </div>
        </div>

        <!-- Telemetry Status Bar -->
        <div class="hidden lg:flex items-center gap-6 font-mono-tech text-xs border-x border-cyan-500/20 px-6">
            <div>
                <span class="text-gray-500 block">STATUS DA REDE</span>
                <span class="text-emerald-400 font-bold flex items-center gap-1">
                    <span class="w-2 h-2 rounded-full bg-emerald-400 animate-ping"></span> OPTICAL LINK ACTIVE
                </span>
            </div>
            <div>
                <span class="text-gray-500 block">PRECISÃO PIEZO</span>
                <span class="text-cyan-400 font-bold" id="telemetry-piezo">&lt; 0.08 µrad</span>
            </div>
            <div>
                <span class="text-gray-500 block">QBER (ERRO QUÂNTICO)</span>
                <span class="text-amber-400 font-bold" id="telemetry-qber">0.42 %</span>
            </div>
            <div>
                <span class="text-gray-500 block">TENSAO BARRAMENTO</span>
                <span class="text-cyan-300 font-bold" id="telemetry-voltage">50.1 VDC</span>
            </div>
        </div>

        <!-- System Actions & Controls -->
        <div class="flex items-center gap-2">
            <button id="btn-audio" class="px-3 py-1.5 rounded hud-panel hover:bg-cyan-500/20 text-cyan-300 flex items-center gap-2 border border-cyan-500/30 transition">
                <i data-lucide="volume-2" class="w-4 h-4"></i> <span class="hidden sm:inline">ÁUDIO</span>
            </button>
            <button id="btn-reset-view" class="px-3 py-1.5 rounded hud-panel hover:bg-cyan-500/20 text-cyan-300 flex items-center gap-2 border border-cyan-500/30 transition">
                <i data-lucide="rotate-ccw" class="w-4 h-4"></i> <span class="hidden sm:inline">RESETA VISÃO</span>
            </button>
        </div>
    </header>

    <!-- Canvas Container (Space Environment & CAD Renderer) -->
    <div class="relative flex-1 my-2 rounded-lg overflow-hidden border border-cyan-500/20 bg-slate-950 flex items-center justify-center">
        <canvas id="srosCanvas" class="w-full h-full cursor-grab active:cursor-grabbing"></canvas>

        <!-- Canvas Floating HUD Overlay (Top-Left View Controls) -->
        <div class="absolute top-4 left-4 z-10 flex flex-col gap-2 max-w-xs">
            <!-- View Selector Tabs -->
            <div class="hud-panel p-1 rounded-lg flex flex-wrap gap-1 border border-cyan-500/30">
                <button data-view="ext" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition bg-cyan-500 text-black shadow-lg shadow-cyan-500/30">
                    Projeção Externa
                </button>
                <button data-view="m1" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20">Mód. 1</button>
                <button data-view="m2" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20">Mód. 2</button>
                <button data-view="m3" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20">Mód. 3</button>
                <button data-view="m4" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20">Mód. 4</button>
                <button data-view="m5" class="view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20">Mód. 5</button>
            </div>

            <!-- View Description Tag -->
            <div class="hud-panel p-2.5 rounded-lg text-xs font-mono-tech border border-cyan-500/20" id="view-desc-panel">
                <div class="text-cyan-400 font-bold mb-1 flex items-center gap-1.5" id="view-title">
                    <i data-lucide="eye" class="w-3.5 h-3.5"></i> VISTA EXTERNA ISO
                </div>
                <p class="text-gray-300 text-[11px] leading-relaxed" id="view-info">
                    Visão geral completa em órbita terrestre. Sistema com malha laser colimada, redundantemente protegido por criptografia quântica.
                </p>
            </div>
        </div>

        <!-- Laser Spectrum Controls (Top Right Overlay) -->
        <div class="absolute top-4 right-4 z-10 hud-panel p-3 rounded-lg border border-cyan-500/30 font-mono-tech space-y-2 text-xs w-48">
            <div class="text-cyan-400 font-bold border-b border-cyan-500/20 pb-1 flex items-center justify-between">
                <span>ESPECTROS ÓPTICOS</span>
                <i data-lucide="zap" class="w-3.5 h-3.5 text-amber-400"></i>
            </div>
            <label class="flex items-center justify-between cursor-pointer text-blue-300 hover:text-white">
                <span class="flex items-center gap-1.5">
                    <span class="w-2.5 h-2.5 rounded-full bg-indigo-500 shadow-[0_0_8px_#6366f1]"></span> 405nm (UV)
                </span>
                <input type="checkbox" id="chk-uv" checked class="accent-indigo-500 cursor-pointer">
            </label>
            <label class="flex items-center justify-between cursor-pointer text-red-300 hover:text-white">
                <span class="flex items-center gap-1.5">
                    <span class="w-2.5 h-2.5 rounded-full bg-red-500 shadow-[0_0_8px_#ef4444]"></span> 650nm (VERMELHO)
                </span>
                <input type="checkbox" id="chk-red" checked class="accent-red-500 cursor-pointer">
            </label>
            <label class="flex items-center justify-between cursor-pointer text-cyan-300 hover:text-white">
                <span class="flex items-center gap-1.5">
                    <span class="w-2.5 h-2.5 rounded-full bg-cyan-400 shadow-[0_0_8px_#22d3ee]"></span> 1550nm (IV)
                </span>
                <input type="checkbox" id="chk-ir" checked class="accent-cyan-400 cursor-pointer">
            </label>
            <div class="pt-2 border-t border-cyan-500/20 space-y-1.5">
                <button id="btn-swarm" class="w-full py-1 px-2 rounded bg-cyan-950 border border-cyan-500/50 hover:bg-cyan-500/20 text-cyan-300 text-[11px] font-bold flex items-center justify-center gap-1 transition">
                    <i data-lucide="grid" class="w-3 h-3"></i> EMITIR MALHA SONDA
                </button>
                <button id="btn-quantum" class="w-full py-1 px-2 rounded bg-amber-950 border border-amber-500/50 hover:bg-amber-500/20 text-amber-300 text-[11px] font-bold flex items-center justify-center gap-1 transition">
                    <i data-lucide="shield-alert" class="w-3 h-3"></i> CHAVE BB84 (PATH A/B)
                </button>
            </div>
        </div>

        <!-- Floating Component Info Overlay (Bottom Right) -->
        <div class="absolute bottom-4 right-4 z-10 hud-panel p-3.5 rounded-lg border border-cyan-500/30 max-w-sm w-full hidden sm:block font-mono-tech">
            <div class="flex items-center justify-between border-b border-cyan-500/20 pb-1.5 mb-2">
                <span class="text-cyan-400 font-bold flex items-center gap-1.5">
                    <i data-lucide="cpu" class="w-4 h-4"></i> ITEM SELECIONADO
                </span>
                <span class="text-xs px-1.5 py-0.5 rounded bg-cyan-950 text-cyan-300 border border-cyan-500/30" id="item-tag">Nenhum</span>
            </div>
            <h3 class="text-sm font-bold text-white mb-1" id="item-title">Clique em um componente</h3>
            <p class="text-xs text-gray-300 leading-relaxed min-h-[48px]" id="item-desc">
                Selecione qualquer elemento listado no menu lateral ou clique nos hotspots técnicos interativos do modelo.
            </p>
            <div class="grid grid-cols-2 gap-2 mt-2 pt-2 border-t border-cyan-500/20 text-[11px]">
                <div><span class="text-gray-500">DIMENSÃO:</span> <span class="text-cyan-300" id="item-dim">-</span></div>
                <div><span class="text-gray-500">TOLERÂNCIA:</span> <span class="text-cyan-300" id="item-tol">-</span></div>
            </div>
        </div>
    </div>

    <!-- Bottom Dashboard Panel & Component Agenda Directory -->
    <footer class="hud-panel hud-border-corner p-3 rounded-lg grid grid-cols-1 md:grid-cols-4 gap-3 z-20">
        <!-- Telemetry Canvas Chart -->
        <div class="flex flex-col justify-between">
            <div class="flex items-center justify-between text-xs font-mono-tech mb-1">
                <span class="text-cyan-400 font-bold flex items-center gap-1">
                    <i data-lucide="activity" class="w-3.5 h-3.5"></i> ESTABILIZAÇÃO DO FEIXE
                </span>
                <span class="text-gray-400" id="chart-val">0.05 µrad</span>
            </div>
            <div class="w-full h-16 bg-slate-900/80 rounded border border-cyan-500/20 overflow-hidden relative">
                <canvas id="telemetryChart" class="w-full h-full"></canvas>
            </div>
        </div>

        <!-- Quick Interactive Component Directory Selector -->
        <div class="md:col-span-3 flex flex-col justify-between">
            <div class="flex items-center justify-between text-xs font-mono-tech mb-1">
                <span class="text-cyan-400 font-bold flex items-center gap-1">
                    <i data-lucide="list" class="w-3.5 h-3.5"></i> AGENDA DESCRITIVA DOS COMPONENTES (NBR / ISO)
                </span>
                <span class="text-gray-400 text-[11px]">CLIQUE PARA DESTACAR</span>
            </div>
            <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-6 gap-1.5 font-mono-tech text-[11px]" id="component-buttons">
                <!-- Buttons injected via JS -->
            </div>
        </div>
    </footer>

    <!-- Main Application JavaScript Logic -->
    <script>
        // --- Technical Specifications Data (Based directly on NBR/ISO Engineering Sheets) ---
        const SROS_DATA = {
            ext: {
                title: "PROJEÇÃO EXTERNA (VISTA GERAL EM ÓRBITA)",
                info: "Sistema SROS em escala 1:20 operando em órbita. Transmissão óptica colimada de ultra alta frequência imune a interferências de RF.",
                items: [
                    { id: 1, name: "Mód. Óptico Principal", legend: "1", dim: "1800 x 900 mm", tol: "± 0.05 mm", desc: "Varredura ativa; telescópio de captação colimado com LIDAR e radar de milímetros." },
                    { id: 2, name: "Segurança Quântica", legend: "2", dim: "700 x 500 mm", tol: "BB84 State", desc: "Criptografia por chave de estado físico, redundância de canal Path A/B." },
                    { id: 3, name: "Gimbals Magnéticos", legend: "3", dim: "Ø 1800 mm", tol: "< 0.1 µrad", desc: "Estabilização por levitação magnética e atuadores piezoelétricos de alta fidelidade angular." },
                    { id: 4, name: "Enxame de Sondas", legend: "4", dim: "Malha IA", tol: "Latência 0ms", desc: "Malha geométrica de lasers de alta densidade operados por inteligência artificial para eliminação de RF." },
                    { id: 5, name: "Feixe Colimado", legend: "5", dim: "Ø 30 mm", tol: "Multi-Freq", desc: "Sincronização e transmissão contínua de dados ópticos de alta frequência." },
                    { id: 6, name: "Telescópios Prim/Sec", legend: "6", dim: "2500 mm span", tol: "Redundante", desc: "Recepção de sinal óptico multi-espectral com ajuste angular de 250°." }
                ]
            },
            m1: {
                title: "MÓDULO 1 - DETALHAMENTO DE ENGENHARIA ÓPTICA",
                info: "Configuração frontal e lateral com canais ópticos multi-espectrais (405nm UV, 650nm Red, 1550nm IR).",
                items: [
                    { id: 1, name: "Módulo Óptico Principal", legend: "1", dim: "1800 mm", tol: "Ø 30 mm", desc: "Lentes de foco colimado primárias com revestimento antirreflexo espacial." },
                    { id: 2, name: "Módulo Óptico Stack", legend: "2", dim: "900 mm", tol: "ISO 128", desc: "Conjunto de empilhamento de lentes secundárias para amplificação coerente." },
                    { id: 3, name: "Liner de Gimbals", legend: "3", dim: "1200 mm", tol: "± 10 mm", desc: "Anel de acoplamento mecânico com isolamento de microvibração." },
                    { id: 4, name: "Multi-Directional Port", legend: "4", dim: "2500 mm", tol: "250° Sweep", desc: "Porta de recepção/emissão multi-direcional articulada." },
                    { id: 5, name: "Recepção Primária", legend: "5", dim: "700 mm", tol: "Multi-Banda", desc: "Telescópio de recepção primário multispectral." },
                    { id: 6, name: "Recepção Secundária", legend: "6", dim: "2x Ø 227mm", tol: "Redundante", desc: "Par de telescópios auxiliares para varredura secundária de enxame." }
                ]
            },
            m2: {
                title: "MÓDULO 2 - SISTEMA DE ESTABILIZAÇÃO E QUÂNTICA",
                info: "Detalhamento D e View C-C do módulo de redundância quântica BB84 e suspensão magnética piezoelétrica.",
                items: [
                    { id: 1, name: "Conjunto Gimbal", legend: "1", dim: "1800 mm", tol: "< 0.1 µrad", desc: "Suspensão magnética ativa sem atrito mecânico para alta precisão." },
                    { id: 2, name: "Módulo Stack 2", legend: "2", dim: "900 mm", tol: "LIDAR", desc: "Unidade de medição LIDAR acoplada para telemetria de distância relativa." },
                    { id: 3, name: "Acionamento Gimbal", legend: "3", dim: "Curso ±10mm", tol: "NBR 10067", desc: "Mecanismo electromagnético de acionamento ultrarrápido." },
                    { id: 7, name: "Atuadores Piezo", legend: "7", dim: "Micron-Scale", tol: "< 0.05 µrad", desc: "Atuadores piezoelétricos para correção óptica em escala nanométrica." },
                    { id: 8, name: "Redundância Quântica", legend: "8", dim: "Path A / B", tol: "BB84 State", desc: "Criptografia quântica por divisão de estado de fóton primário/redundante." },
                    { id: 9, name: "Bobina Magnética", legend: "9", dim: "High-Inductance", tol: "Mag-Lev", desc: "Bobina magnética de suspensão e estabilização de feixe laser." }
                ]
            },
            m3: {
                title: "MÓDULO 3 - COLIMADOR E MALHA DE SONDA IA",
                info: "Sistema de emissão Probe Swarm Laser Mesh e óptica multispectral de alta potência.",
                items: [
                    { id: 1, name: "Gimbal de Suspensão", legend: "1", dim: "1800 mm", tol: "Piezo Dual", desc: "Gimbal completo de estabilização de malha óptico-dinâmica." },
                    { id: 2, name: "Módulo Óptico Stack", legend: "2", dim: "900 mm", tol: "ISO 1126", desc: "Colimador de alta frequência para sinal contínuo." },
                    { id: 6, name: "Telescópios Multi-Esp", legend: "6", dim: "405/650/1550nm", tol: "Band-Pass", desc: "Recepção espectral combinada UV, Visível e Infravermelho." },
                    { id: 7, name: "Atuadores Piezo", legend: "7", dim: "Fine-Pitch", tol: "< 0.1 µrad", desc: "Controle piezoelétrico fino de curvatura focal." },
                    { id: 10, name: "Telescópios Multispectrais", legend: "10", dim: "Dual-Lens", tol: "NBR 10126", desc: "Conjunto primário de lentes multispectrais para longa distância." },
                    { id: 11, name: "Malha Laser Sonda IA", legend: "11", dim: "Grid Density", tol: "IA Dynamic", desc: "Emissor de lasers interligados operados por inteligência artificial para enxames de microssatélites." }
                ]
            },
            m4: {
                title: "MÓDULO 4 - SISTEMA DE POTÊNCIA E TELEMETRIA",
                info: "Painel eletrônico, conversor DC-DC multi-canal, capacitores e unidade RTU/TM/TC.",
                items: [
                    { id: 1, name: "Conversores DC-DC", legend: "1", dim: "100-250V AC/DC", tol: "Eff > 92%", desc: "Múltiplos canais regulados (28V, 50V DC) com comutação a 100 kHz." },
                    { id: 2, name: "Unidade de Telemetria", legend: "2", dim: "RTU / TM / TC", tol: "Memória Massa", desc: "Processamento de dados da plataforma e telemetria de controle óptico." },
                    { id: 3, name: "Controlador OBC", legend: "3", dim: "Barramento", tol: "Manifold Opt", desc: "Interface do computador de bordo e gerenciamento do fluxo óptico." },
                    { id: 4, name: "Capacitores Alta Tensão", legend: "4", dim: "Reserve-Bank", tol: "12 kW Surge", desc: "Reserva de energia rápida para disparo do enxame de lasers e pulsos BB84." },
                    { id: 7, name: "Atuadores de Apontamento", legend: "7", dim: "Direct Drive", tol: "0.01° Step", desc: "Interface de atuadores acoplados diretamente ao barramento de energia." },
                    { id: 10, name: "Receptores Multispectrais", legend: "10", dim: "Bus-Linked", tol: "Opt-Coupled", desc: "Acopladores ópticos de alta velocidade para comunicação interna." }
                ]
            },
            m5: {
                title: "MÓDULO 5 - ESTRUTURA E MATRIZ DE MANUTENÇÃO",
                info: "Chassi estrutural completo, conectores umbilicais, mantas MLI e especificação de manutenção NBR/ISO.",
                items: [
                    { id: 1, name: "Flange de Montagem", legend: "1", dim: "4x Ø 30 mm", tol: "ISO 128", desc: "Pontos de fixação estrutural reforçados para o módulo satelital." },
                    { id: 2, name: "Reforço Estrutural", legend: "2", dim: "30 mm Thick", tol: "Titanium Gr5", desc: "Estrutura de suporte primária resistente a cargas de lançamento." },
                    { id: 6, name: "Trilho de Montagem", legend: "6", dim: "1200 mm Base", tol: "± 0.2 mm", desc: "Trilhos de alinhamento de precisão para acoplamento do módulo." },
                    { id: 7, name: "Estrutura Inferior", legend: "7", dim: "1200 mm", tol: "Structural", desc: "Chassi mecânico inferior de absorção de vibração." },
                    { id: 9, name: "Conectores Umbilicais", legend: "9", dim: "Multi-Pin", tol: "MIL-STD", desc: "Interface de energia, dados ópticos e linhas de redundância." },
                    { id: 11, name: "Amortecedores Vibração", legend: "11", dim: "Elastomeric", tol: "Damping ISO", desc: "Pads de isolamento mecânico contra frequências harmônicas." }
                ]
            }
        };

        // --- Application State ---
        let currentView = 'ext';
        let selectedItem = null;
        let isAudioMuted = true;
        let audioCtx = null;
        let zoomLevel = 1.0;
        let panOffset = { x: 0, y: 0 };
        let isDragging = false;
        let dragStart = { x: 0, y: 0 };
        let quantumModeActive = false;
        let swarmActive = true;

        // Laser Spectrum Toggles
        const laserStates = {
            uv: true,
            red: true,
            ir: true
        };

        // --- Telemetry Simulated Data History ---
        const telemetryHistory = new Array(40).fill(0.05);

        // --- DOM Elements ---
        const canvas = document.getElementById('srosCanvas');
        const ctx = canvas.getContext('2d');
        const telemetryCanvas = document.getElementById('telemetryChart');
        const telemetryCtx = telemetryCanvas.getContext('2d');

        // UI Text Elements
        const viewTitle = document.getElementById('view-title');
        const viewInfo = document.getElementById('view-info');
        const itemTag = document.getElementById('item-tag');
        const itemTitle = document.getElementById('item-title');
        const itemDesc = document.getElementById('item-desc');
        const itemDim = document.getElementById('item-dim');
        const itemTol = document.getElementById('item-tol');
        const componentButtonsContainer = document.getElementById('component-buttons');

        // Telemetry Text
        const telPiezo = document.getElementById('telemetry-piezo');
        const telQber = document.getElementById('telemetry-qber');
        const telVoltage = document.getElementById('telemetry-voltage');
        const chartVal = document.getElementById('chart-val');

        // --- Canvas Resizing ---
        function resizeCanvas() {
            const rect = canvas.parentElement.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;

            const tRect = telemetryCanvas.parentElement.getBoundingClientRect();
            telemetryCanvas.width = tRect.width;
            telemetryCanvas.height = tRect.height;
        }
        window.addEventListener('resize', resizeCanvas);

        // --- Web Audio Synthesizer (Space Ambient Sound & Effects) ---
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }

        function playSound(type) {
            if (isAudioMuted || !audioCtx) return;
            try {
                const now = audioCtx.currentTime;
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();

                if (type === 'click') {
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(800, now);
                    osc.frequency.exponentialRampToValueAtTime(400, now + 0.05);
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.05);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now);
                    osc.stop(now + 0.05);
                } else if (type === 'laser') {
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(1200, now);
                    osc.frequency.exponentialRampToValueAtTime(200, now + 0.2);
                    gain.gain.setValueAtTime(0.1, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.2);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now);
                    osc.stop(now + 0.2);
                } else if (type === 'quantum') {
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(300, now);
                    osc.frequency.linearRampToValueAtTime(900, now + 0.3);
                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now);
                    osc.stop(now + 0.3);
                }
            } catch (e) {
                console.warn("Audio error:", e);
            }
        }

        // --- Starfield & Background Generation ---
        const stars = Array.from({ length: 150 }, () => ({
            x: Math.random(),
            y: Math.random(),
            size: Math.random() * 1.5 + 0.5,
            alpha: Math.random() * 0.8 + 0.2,
            speed: Math.random() * 0.0002 + 0.00005
        }));

        function drawBackground() {
            const w = canvas.width;
            const h = canvas.height;

            // Deep space gradient
            const bgGradient = ctx.createRadialGradient(w/2, h/2, 50, w/2, h/2, Math.max(w, h));
            bgGradient.addColorStop(0, '#090d16');
            bgGradient.addColorStop(0.6, '#04070d');
            bgGradient.addColorStop(1, '#020305');
            ctx.fillStyle = bgGradient;
            ctx.fillRect(0, 0, w, h);

            // Stars animation
            ctx.save();
            stars.forEach(star => {
                star.y += star.speed;
                if (star.y > 1) star.y = 0;
                ctx.fillStyle = `rgba(255, 255, 255, ${star.alpha})`;
                ctx.beginPath();
                ctx.arc(star.x * w, star.y * h, star.size, 0, Math.PI * 2);
                ctx.fill();
            });

            // Distant Earth Atmosphere Curve (Bottom Left)
            const earthX = -w * 0.2;
            const earthY = h * 1.4;
            const earthR = Math.min(w, h) * 1.1;

            const earthGlow = ctx.createRadialGradient(earthX, earthY, earthR * 0.8, earthX, earthY, earthR * 1.15);
            earthGlow.addColorStop(0, 'rgba(15, 45, 90, 0.8)');
            earthGlow.addColorStop(0.75, 'rgba(0, 180, 255, 0.25)');
            earthGlow.addColorStop(0.9, 'rgba(0, 243, 255, 0.1)');
            earthGlow.addColorStop(1, 'rgba(0, 0, 0, 0)');

            ctx.fillStyle = earthGlow;
            ctx.beginPath();
            ctx.arc(earthX, earthY, earthR * 1.15, 0, Math.PI * 2);
            ctx.fill();

            // Earth Limb Arc Line
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.4)';
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.arc(earthX, earthY, earthR, -Math.PI * 0.4, 0);
            ctx.stroke();

            ctx.restore();
        }

        // --- Technical Drawing Helpers (Canvas 2D Blueprint Renderer) ---
        function drawGrid(cx, cy, size, ticks) {
            ctx.save();
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.07)';
            ctx.lineWidth = 1;
            const half = (size * ticks) / 2;

            for (let i = -ticks; i <= ticks; i++) {
                // Horizontal line
                ctx.beginPath();
                ctx.moveTo(cx - half, cy + i * size);
                ctx.lineTo(cx + half, cy + i * size);
                ctx.stroke();

                // Vertical line
                ctx.beginPath();
                ctx.moveTo(cx + i * size, cy - half);
                ctx.lineTo(cx + i * size, cy + half);
                ctx.stroke();
            }
            ctx.restore();
        }

        function drawDimensionLine(x1, y1, x2, y2, text) {
            ctx.save();
            ctx.strokeStyle = '#00f3ff';
            ctx.fillStyle = '#00f3ff';
            ctx.lineWidth = 1;
            ctx.font = '10px "Share Tech Mono"';

            // Main line
            ctx.beginPath();
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.stroke();

            // End ticks
            const angle = Math.atan2(y2 - y1, x2 - x1);
            const perp = angle + Math.PI / 2;
            const tickLen = 4;

            [ [x1, y1], [x2, y2] ].forEach(([x, y]) => {
                ctx.beginPath();
                ctx.moveTo(x + Math.cos(perp)*tickLen, y + Math.sin(perp)*tickLen);
                ctx.lineTo(x - Math.cos(perp)*tickLen, y - Math.sin(perp)*tickLen);
                ctx.stroke();
            });

            // Text
            if (text) {
                const midX = (x1 + x2) / 2;
                const midY = (y1 + y2) / 2;
                ctx.save();
                ctx.translate(midX, midY);
                ctx.fillStyle = '#00f3ff';
                ctx.fillRect(-20, -7, 40, 12);
                ctx.fillStyle = '#030712';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(text, 0, 0);
                ctx.restore();
            }
            ctx.restore();
        }

        // --- Satellite Renderers ---

        // Render External View (Projeção Externa Isometric / Satellite Orbit View)
        function drawExternalSatellite(time) {
            ctx.save();

            // Hover / Rotation pulse
            const floatOffset = Math.sin(time * 0.0015) * 6;
            ctx.translate(0, floatOffset);

            // Tech Shadow
            ctx.fillStyle = 'rgba(0, 243, 255, 0.05)';
            ctx.beginPath();
            ctx.ellipse(0, 140, 180, 40, 0, 0, Math.PI * 2);
            ctx.fill();

            // 1. Solar Panels (Left & Right Wings)
            ctx.strokeStyle = '#00f3ff';
            ctx.lineWidth = 1.5;

            // Left Solar Panel
            ctx.fillStyle = 'rgba(15, 30, 60, 0.8)';
            ctx.beginPath();
            ctx.moveTo(-100, -10);
            ctx.lineTo(-240, -90);
            ctx.lineTo(-210, -150);
            ctx.lineTo(-70, -70);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // Left Solar Grid lines
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.3)';
            ctx.beginPath();
            ctx.moveTo(-170, -50); ctx.lineTo(-140, -110);
            ctx.moveTo(-135, -30); ctx.lineTo(-105, -90);
            ctx.stroke();

            // Right Solar Panel
            ctx.strokeStyle = '#00f3ff';
            ctx.fillStyle = 'rgba(15, 30, 60, 0.8)';
            ctx.beginPath();
            ctx.moveTo(80, 40);
            ctx.lineTo(220, 120);
            ctx.lineTo(190, 180);
            ctx.lineTo(50, 100);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // 2. Satellite Main Chassis Frame (Cube Isometric)
            const isSelectedMain = selectedItem === 1;
            ctx.fillStyle = isSelectedMain ? 'rgba(0, 243, 255, 0.3)' : 'rgba(20, 30, 50, 0.85)';
            ctx.strokeStyle = isSelectedMain ? '#00ff88' : '#00f3ff';
            ctx.lineWidth = isSelectedMain ? 2.5 : 1.5;

            // Chassis Front Face
            ctx.beginPath();
            ctx.moveTo(-60, -40);
            ctx.lineTo(40, 10);
            ctx.lineTo(40, 90);
            ctx.lineTo(-60, 40);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // Chassis Top Face
            ctx.beginPath();
            ctx.moveTo(-60, -40);
            ctx.lineTo(40, 10);
            ctx.lineTo(100, -25);
            ctx.lineTo(0, -75);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // Chassis Side Face
            ctx.beginPath();
            ctx.moveTo(40, 10);
            ctx.lineTo(100, -25);
            ctx.lineTo(100, 55);
            ctx.lineTo(40, 90);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // 3. Main Telescope Lens Cylinder (Módulo Óptico Principal)
            const isSelectedTelescope = selectedItem === 1 || selectedItem === 5;
            ctx.save();
            ctx.translate(-90, 25);
            ctx.rotate(-Math.PI / 6);

            // Lens Outer Barrel
            ctx.fillStyle = isSelectedTelescope ? 'rgba(0, 255, 136, 0.2)' : 'rgba(10, 20, 40, 0.9)';
            ctx.strokeStyle = isSelectedTelescope ? '#00ff88' : '#00f3ff';
            ctx.lineWidth = 2;

            ctx.beginPath();
            ctx.ellipse(-30, 0, 25, 45, 0, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            ctx.beginPath();
            ctx.rect(-30, -45, 70, 90);
            ctx.fill();
            ctx.stroke();

            // Lens Front Aperture Ring
            ctx.fillStyle = 'rgba(0, 243, 255, 0.3)';
            ctx.beginPath();
            ctx.ellipse(40, 0, 25, 45, 0, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            // Lens Concentric Rings (Detail)
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.6)';
            ctx.beginPath();
            ctx.ellipse(40, 0, 15, 30, 0, 0, Math.PI * 2);
            ctx.stroke();

            ctx.restore();

            // 4. Quantum Redundancy & Gimbal Elements
            const isSelectedGimbal = selectedItem === 3;
            if (isSelectedGimbal) {
                ctx.strokeStyle = '#ffaa00';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(0, 0, 110, 0, Math.PI * 2);
                ctx.setLineDash([4, 4]);
                ctx.stroke();
                ctx.setLineDash([]);
            }

            // 5. Laser Beams Emission Simulation (From Telescope Aperture)
            const beamStartX = -135;
            const beamStartY = 50;

            if (laserStates.uv) {
                // 405nm Ultraviolet Laser
                ctx.strokeStyle = 'rgba(99, 102, 241, 0.8)';
                ctx.shadowColor = '#6366f1';
                ctx.shadowBlur = 12;
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(beamStartX, beamStartY - 5);
                ctx.lineTo(-400, beamStartY - 80);
                ctx.stroke();
            }

            if (laserStates.red) {
                // 650nm Red Laser
                ctx.strokeStyle = 'rgba(239, 68, 68, 0.9)';
                ctx.shadowColor = '#ef4444';
                ctx.shadowBlur = 15;
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(beamStartX, beamStartY);
                ctx.lineTo(-400, beamStartY - 50);
                ctx.stroke();
            }

            if (laserStates.ir) {
                // 1550nm Infrared Laser (Pulsing Cyan)
                const pulse = (Math.sin(time * 0.008) + 1) * 0.5;
                ctx.strokeStyle = `rgba(34, 211, 238, ${0.4 + pulse * 0.5})`;
                ctx.shadowColor = '#22d3ee';
                ctx.shadowBlur = 20;
                ctx.lineWidth = 3.5;
                ctx.beginPath();
                ctx.moveTo(beamStartX, beamStartY + 5);
                ctx.lineTo(-400, beamStartY - 20);
                ctx.stroke();
            }
            ctx.shadowBlur = 0; // Reset shadow

            // 6. Quantum Key Protocol Particles (BB84 Channel Simulation)
            if (quantumModeActive) {
                ctx.save();
                const particleCount = 12;
                for (let i = 0; i < particleCount; i++) {
                    const progress = ((time * 0.001 + i / particleCount) % 1);
                    const px = beamStartX - progress * 265;
                    const py = beamStartY - progress * 40 + Math.sin(progress * Math.PI * 4) * 6;

                    ctx.fillStyle = i % 2 === 0 ? '#ffaa00' : '#00ff88';
                    ctx.shadowColor = ctx.fillStyle;
                    ctx.shadowBlur = 8;
                    ctx.beginPath();
                    ctx.arc(px, py, 3, 0, Math.PI * 2);
                    ctx.fill();

                    // Quantum bit state label
                    ctx.fillStyle = '#ffffff';
                    ctx.font = '8px "Share Tech Mono"';
                    ctx.fillText(i % 2 === 0 ? '|ψ⟩' : '|1⟩', px - 4, py - 6);
                }
                ctx.restore();
            }

            // 7. Probe Swarm Laser Mesh (Enxame de Sondas)
            if (swarmActive) {
                ctx.save();
                ctx.strokeStyle = 'rgba(0, 243, 255, 0.35)';
                ctx.lineWidth = 1;

                const swarmNodes = [
                    { x: -280, y: -180 },
                    { x: -340, y: -90 },
                    { x: -310, y: 80 },
                    { x: -220, y: 150 },
                    { x: -160, y: -140 }
                ];

                // Draw Probe Satellite Nodes
                swarmNodes.forEach((node, idx) => {
                    // Node oscillation
                    const nx = node.x + Math.sin(time * 0.002 + idx) * 8;
                    const ny = node.y + Math.cos(time * 0.002 + idx) * 8;

                    // Micro Probe Symbol
                    ctx.fillStyle = '#00f3ff';
                    ctx.beginPath();
                    ctx.arc(nx, ny, 3, 0, Math.PI * 2);
                    ctx.fill();

                    // Interconnecting Mesh Lines
                    swarmNodes.forEach((otherNode, oIdx) => {
                        if (idx < oIdx) {
                            const onx = otherNode.x + Math.sin(time * 0.002 + oIdx) * 8;
                            const ony = otherNode.y + Math.cos(time * 0.002 + oIdx) * 8;
                            ctx.beginPath();
                            ctx.moveTo(nx, ny);
                            ctx.lineTo(onx, ony);
                            ctx.stroke();
                        }
                    });

                    // Beam to Satellite Aperture
                    ctx.strokeStyle = 'rgba(0, 255, 136, 0.25)';
                    ctx.beginPath();
                    ctx.moveTo(nx, ny);
                    ctx.lineTo(beamStartX, beamStartY);
                    ctx.stroke();
                });
                ctx.restore();
            }

            // Technical Annotations & ISO Dimensions Overlay
            drawDimensionLine(-240, -110, 80, 20, "2500 mm");
            drawDimensionLine(-60, 60, 40, 110, "1800 mm");

            ctx.restore();
        }

        // Render Internal Blueprint Modules (Módulos 1 a 5 Engineering Projection)
        function drawInternalModule(moduleKey, time) {
            ctx.save();

            // Technical Engineering Grid Background
            drawGrid(0, 0, 30, 12);

            // Main Blueprint Container Box (ISO Standard Front / Side Cut View)
            ctx.strokeStyle = '#00f3ff';
            ctx.lineWidth = 2;
            ctx.fillStyle = 'rgba(10, 20, 35, 0.85)';

            // Left Section (Front Elevation)
            ctx.beginPath();
            ctx.rect(-220, -140, 200, 280);
            ctx.fill();
            ctx.stroke();

            // Right Section (Side Cut Section)
            ctx.beginPath();
            ctx.rect(20, -140, 200, 280);
            ctx.fill();
            ctx.stroke();

            // Section Divider Center Axis Lines (Dash-Dot ISO Standard)
            ctx.strokeStyle = 'rgba(255, 50, 50, 0.5)';
            ctx.lineWidth = 1;
            ctx.setLineDash([12, 3, 2, 3]);

            ctx.beginPath();
            ctx.moveTo(-120, -170); ctx.lineTo(-120, 170); // Left Center Axis
            ctx.moveTo(120, -170); ctx.lineTo(120, 170);   // Right Center Axis
            ctx.moveTo(-250, 0); ctx.lineTo(250, 0);       // Horizontal Center Axis
            ctx.stroke();
            ctx.setLineDash([]); // Reset line dash

            // --- Module-Specific Internal Details ---
            if (moduleKey === 'm1') {
                // Módulo 1: Main Optics Aperture Stack
                ctx.strokeStyle = '#00f3ff';
                ctx.fillStyle = 'rgba(0, 243, 255, 0.15)';

                // Front Lens Rings
                for (let r = 80; r >= 20; r -= 20) {
                    ctx.beginPath();
                    ctx.arc(-120, 0, r, 0, Math.PI * 2);
                    ctx.stroke();
                }

                // Side Cut Optics Array
                ctx.fillStyle = 'rgba(0, 255, 136, 0.2)';
                ctx.beginPath();
                ctx.rect(50, -60, 30, 120);
                ctx.rect(100, -80, 40, 160);
                ctx.rect(160, -40, 25, 80);
                ctx.fill();
                ctx.stroke();

                // Rays inside side cut
                ctx.strokeStyle = 'rgba(239, 68, 68, 0.7)';
                ctx.beginPath();
                ctx.moveTo(20, -100); ctx.lineTo(120, -20); ctx.lineTo(200, 0);
                ctx.moveTo(20, 100); ctx.lineTo(120, 20); ctx.lineTo(200, 0);
                ctx.stroke();

            } else if (moduleKey === 'm2') {
                // Módulo 2: Gimbal & Quantum BB84 Redundancy
                ctx.strokeStyle = '#ffaa00';

                // Piezo Actuator Coils
                ctx.fillStyle = 'rgba(255, 170, 0, 0.2)';
                ctx.beginPath();
                ctx.rect(-180, -100, 40, 60);
                ctx.rect(-180, 40, 40, 60);
                ctx.rect(-60, -100, 40, 60);
                ctx.rect(-60, 40, 40, 60);
                ctx.fill();
                ctx.stroke();

                // Quantum Prism Redundancy Schematic (Bottom Left)
                ctx.save();
                ctx.translate(120, 0);
                ctx.strokeStyle = '#00ff88';
                ctx.fillStyle = 'rgba(0, 255, 136, 0.15)';

                // Path A / B Prisms
                ctx.beginPath();
                ctx.moveTo(-40, -40); ctx.lineTo(0, -70); ctx.lineTo(40, -40); ctx.closePath();
                ctx.moveTo(-40, 40); ctx.lineTo(0, 70); ctx.lineTo(40, 40); ctx.closePath();
                ctx.fill();
                ctx.stroke();

                // Quantum Paths
                ctx.strokeStyle = '#00f3ff';
                ctx.beginPath();
                ctx.moveTo(-80, -50); ctx.lineTo(0, -50); ctx.lineTo(80, 50);
                ctx.moveTo(-80, 50); ctx.lineTo(0, 50); ctx.lineTo(80, -50);
                ctx.stroke();

                ctx.restore();

            } else if (moduleKey === 'm3') {
                // Módulo 3: Probe Swarm Collimator & Multispectral Array
                ctx.strokeStyle = '#00ff88';

                // Mesh Matrix Emitter
                ctx.fillStyle = 'rgba(0, 255, 136, 0.1)';
                ctx.beginPath();
                ctx.arc(-120, 0, 70, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                // Dots Matrix
                for (let x = -160; x <= -80; x += 15) {
                    for (let y = -40; y <= 40; y += 15) {
                        ctx.fillStyle = '#00ff88';
                        ctx.beginPath();
                        ctx.arc(x, y, 2, 0, Math.PI * 2);
                        ctx.fill();
                    }
                }

                // Side Cut Multi-Angle Angular Sweep Indicators
                ctx.strokeStyle = 'rgba(0, 243, 255, 0.6)';
                ctx.beginPath();
                ctx.arc(120, 0, 100, -Math.PI * 0.35, Math.PI * 0.35);
                ctx.stroke();
                ctx.font = '10px "Share Tech Mono"';
                ctx.fillStyle = '#00f3ff';
                ctx.fillText('250° SWEEP', 170, -105);

            } else if (moduleKey === 'm4') {
                // Módulo 4: Power System & Bus Interface
                ctx.strokeStyle = '#00f3ff';

                // PCB & Capacitor Bank Block (Front View)
                ctx.fillStyle = 'rgba(15, 25, 45, 0.9)';
                ctx.beginPath();
                ctx.rect(-200, -120, 160, 240);
                ctx.fill();
                ctx.stroke();

                // High Voltage Capacitors Array
                for (let i = 0; i < 4; i++) {
                    ctx.fillStyle = 'rgba(255, 170, 0, 0.3)';
                    ctx.strokeStyle = '#ffaa00';
                    ctx.beginPath();
                    ctx.arc(-170 + (i % 2) * 50, -70 + Math.floor(i / 2) * 60, 18, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.stroke();
                }

                // Telemetry Ports (Side Cut)
                ctx.fillStyle = 'rgba(0, 243, 255, 0.2)';
                ctx.strokeStyle = '#00f3ff';
                for (let y = -100; y <= 80; y += 30) {
                    ctx.beginPath();
                    ctx.rect(50, y, 35, 18);
                    ctx.fill();
                    ctx.stroke();
                }

            } else if (moduleKey === 'm5') {
                // Módulo 5: Chassis Structural Rail & ISO Maintenance
                ctx.strokeStyle = '#00f3ff';

                // Mounting Flanges (4x Ø 30mm)
                ctx.fillStyle = 'rgba(0, 243, 255, 0.3)';
                [[-200, -120], [-40, -120], [-200, 120], [-40, 120]].forEach(([fx, fy]) => {
                    ctx.beginPath();
                    ctx.arc(fx, fy, 12, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.stroke();
                });

                // Bottom Mounting Rail (1200mm Base)
                ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
                ctx.beginPath();
                ctx.rect(-210, 125, 180, 15);
                ctx.rect(30, 125, 180, 15);
                ctx.fill();
                ctx.stroke();
            }

            // Dimensions Overlay
            drawDimensionLine(-220, 155, -20, 155, "1800 mm");
            drawDimensionLine(20, 155, 220, 155, "1200 mm");
            drawDimensionLine(235, -140, 235, 140, "2500 mm");

            ctx.restore();
        }

        // --- Main Canvas Render Loop ---
        function render(time) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 1. Render Dynamic Space Background
            drawBackground();

            // 2. Render Interactive CAD Model / View System
            ctx.save();

            // Center Origin + Pan & Zoom
            ctx.translate(canvas.width / 2 + panOffset.x, canvas.height / 2 + panOffset.y);
            ctx.scale(zoomLevel, zoomLevel);

            if (currentView === 'ext') {
                drawExternalSatellite(time);
            } else {
                drawInternalModule(currentView, time);
            }

            ctx.restore();

            // Request next frame
            requestAnimationFrame(render);
        }

        // --- Telemetry Chart Render (Bottom Left Hud Graph) ---
        function updateTelemetryChart() {
            // Add new random variation to piezo alignment (<0.1 µrad)
            const newVal = 0.04 + Math.random() * 0.05;
            telemetryHistory.push(newVal);
            telemetryHistory.shift();

            // Update DOM telemetry indicators
            telPiezo.textContent = `< ${newVal.toFixed(3)} µrad`;
            telQber.textContent = `${(0.35 + Math.random() * 0.15).toFixed(2)} %`;
            telVoltage.textContent = `${(50.0 + (Math.random() - 0.5) * 0.4).toFixed(1)} VDC`;
            chartVal.textContent = `${newVal.toFixed(3)} µrad`;

            // Draw Telemetry Graph Canvas
            const w = telemetryCanvas.width;
            const h = telemetryCanvas.height;
            telemetryCtx.clearRect(0, 0, w, h);

            // Background grid
            telemetryCtx.strokeStyle = 'rgba(0, 243, 255, 0.1)';
            telemetryCtx.lineWidth = 1;
            for (let x = 0; x < w; x += 20) {
                telemetryCtx.beginPath();
                telemetryCtx.moveTo(x, 0); telemetryCtx.lineTo(x, h);
                telemetryCtx.stroke();
            }

            // Plot Telemetry Line
            telemetryCtx.strokeStyle = '#00f3ff';
            telemetryCtx.lineWidth = 1.5;
            telemetryCtx.beginPath();

            const step = w / (telemetryHistory.length - 1);
            telemetryHistory.forEach((val, idx) => {
                const normY = h - (val / 0.15) * h;
                if (idx === 0) telemetryCtx.moveTo(0, normY);
                else telemetryCtx.lineTo(idx * step, normY);
            });
            telemetryCtx.stroke();

            // Area fill under graph
            telemetryCtx.lineTo(w, h);
            telemetryCtx.lineTo(0, h);
            telemetryCtx.fillStyle = 'rgba(0, 243, 255, 0.08)';
            telemetryCtx.fill();
        }

        setInterval(updateTelemetryChart, 400);

        // --- UI Interactions & Event Handlers ---

        // View Switching Logic
        function setView(viewKey) {
            currentView = viewKey;
            const viewData = SROS_DATA[viewKey];

            // Update View Header Text
            viewTitle.innerHTML = `<i data-lucide="eye" class="w-3.5 h-3.5"></i> ${viewData.title}`;
            viewInfo.textContent = viewData.info;
            lucide.createIcons();

            // Highlight Active View Button
            document.querySelectorAll('.view-btn').forEach(btn => {
                if (btn.dataset.view === viewKey) {
                    btn.className = "view-btn px-2.5 py-1 text-xs rounded font-semibold transition bg-cyan-500 text-black shadow-lg shadow-cyan-500/30";
                } else {
                    btn.className = "view-btn px-2.5 py-1 text-xs rounded font-semibold transition text-cyan-300 hover:bg-cyan-500/20";
                }
            });

            // Populate Agenda Component Directory Buttons for current view
            populateComponentButtons(viewData.items);

            // Select first item by default
            selectComponent(viewData.items[0]);
            playSound('click');
        }

        // Component Selection & Info Overlay Update
        function selectComponent(item) {
            selectedItem = item.id;

            itemTag.textContent = `ITEM ${item.legend}`;
            itemTitle.textContent = item.name;
            itemDesc.textContent = item.desc;
            itemDim.textContent = item.dim;
            itemTol.textContent = item.tol;

            // Update button styles
            document.querySelectorAll('.comp-btn').forEach(btn => {
                if (parseInt(btn.dataset.id) === item.id) {
                    btn.className = "comp-btn p-1.5 rounded bg-cyan-500/30 border border-cyan-400 text-cyan-300 font-bold truncate text-left";
                } else {
                    btn.className = "comp-btn p-1.5 rounded bg-slate-900/60 border border-cyan-500/20 text-gray-400 hover:text-cyan-300 truncate text-left";
                }
            });

            playSound('click');
        }

        function populateComponentButtons(items) {
            componentButtonsContainer.innerHTML = '';
            items.forEach(item => {
                const btn = document.createElement('button');
                btn.dataset.id = item.id;
                btn.className = "comp-btn p-1.5 rounded bg-slate-900/60 border border-cyan-500/20 text-gray-400 hover:text-cyan-300 truncate text-left";
                btn.innerHTML = `<span class="text-cyan-400 font-bold mr-1">[${item.legend}]</span> ${item.name}`;
                btn.addEventListener('click', () => selectComponent(item));
                componentButtonsContainer.appendChild(btn);
            });
        }

        // View Tab Button Listeners
        document.querySelectorAll('.view-btn').forEach(btn => {
            btn.addEventListener('click', () => setView(btn.dataset.view));
        });

        // Laser Checkboxes
        document.getElementById('chk-uv').addEventListener('change', (e) => {
            laserStates.uv = e.target.checked;
            playSound('laser');
        });
        document.getElementById('chk-red').addEventListener('change', (e) => {
            laserStates.red = e.target.checked;
            playSound('laser');
        });
        document.getElementById('chk-ir').addEventListener('change', (e) => {
            laserStates.ir = e.target.checked;
            playSound('laser');
        });

        // Swarm Laser Mesh Toggle Button
        document.getElementById('btn-swarm').addEventListener('click', () => {
            swarmActive = !swarmActive;
            playSound('laser');
        });

        // BB84 Quantum Channel Key Switch Button
        document.getElementById('btn-quantum').addEventListener('click', () => {
            quantumModeActive = !quantumModeActive;
            playSound('quantum');
        });

        // Audio Toggle
        document.getElementById('btn-audio').addEventListener('click', () => {
            initAudio();
            isAudioMuted = !isAudioMuted;
            const btn = document.getElementById('btn-audio');
            btn.innerHTML = isAudioMuted 
                ? `<i data-lucide="volume-x" class="w-4 h-4"></i> <span class="hidden sm:inline">MUTE</span>`
                : `<i data-lucide="volume-2" class="w-4 h-4"></i> <span class="hidden sm:inline">ÁUDIO</span>`;
            lucide.createIcons();
            playSound('click');
        });

        // Reset View Pan & Zoom
        document.getElementById('btn-reset-view').addEventListener('click', () => {
            zoomLevel = 1.0;
            panOffset = { x: 0, y: 0 };
            playSound('click');
        });

        // --- Mouse Pan & Zoom Interaction on Canvas ---
        canvas.addEventListener('mousedown', (e) => {
            isDragging = true;
            dragStart = { x: e.clientX - panOffset.x, y: e.clientY - panOffset.y };
        });

        window.addEventListener('mousemove', (e) => {
            if (isDragging) {
                panOffset.x = e.clientX - dragStart.x;
                panOffset.y = e.clientY - dragStart.y;
            }
        });

        window.addEventListener('mouseup', () => {
            isDragging = false;
        });

        canvas.addEventListener('wheel', (e) => {
            e.preventDefault();
            const zoomFactor = e.deltaY < 0 ? 1.1 : 0.9;
            zoomLevel = Math.min(Math.max(0.5, zoomLevel * zoomFactor), 3.0);
        }, { passive: false });

        // Touch support for mobile devices
        let touchStartDist = 0;
        canvas.addEventListener('touchstart', (e) => {
            if (e.touches.length === 1) {
                isDragging = true;
                dragStart = { x: e.touches[0].clientX - panOffset.x, y: e.touches[0].clientY - panOffset.y };
            } else if (e.touches.length === 2) {
                touchStartDist = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
            }
        });

        canvas.addEventListener('touchmove', (e) => {
            if (isDragging && e.touches.length === 1) {
                panOffset.x = e.touches[0].clientX - dragStart.x;
                panOffset.y = e.touches[0].clientY - dragStart.y;
            } else if (e.touches.length === 2) {
                const dist = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
                const zoomFactor = dist / touchStartDist;
                zoomLevel = Math.min(Math.max(0.5, zoomLevel * zoomFactor), 3.0);
                touchStartDist = dist;
            }
        });

        canvas.addEventListener('touchend', () => {
            isDragging = false;
        });

        // --- Initialization ---
        window.onload = () => {
            resizeCanvas();
            lucide.createIcons();
            setView('ext');
            requestAnimationFrame(render);
        };
    </script>
</body>
</html>
