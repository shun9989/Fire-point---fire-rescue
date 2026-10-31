# Fire-point---fire-rescue
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>火線任務（直覺操作升級版）</title>
    <!-- Tailwind CSS for Modern Styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js for 3D Graphics -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { margin: 0; overflow: hidden; font-family: system-ui, -apple-system, sans-serif; background-color: #0f172a; color: #f8fafc; }
        #canvas-container { width: 100vw; height: 100vh; position: absolute; top: 0; left: 0; z-index: 1; }
        .glass-panel { background: rgba(15, 23, 42, 0.85); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); }
    </style>
</head>
<body class="relative">

    <!-- 3D 渲染容器 -->
    <div id="canvas-container"></div>

    <!-- 頂部導覽列：品牌與狀態 -->
    <header class="absolute top-0 left-0 right-0 z-20 flex justify-between items-center p-4 glass-panel border-b">
        <div class="flex items-center space-x-3">
            <span class="bg-red-600 text-white font-bold p-2 rounded-lg shadow-lg"><i class="fa-solid fa-fire-flame-curved"></i></span>
            <div>
                <h1 class="text-lg font-bold tracking-wider">火線任務・消防救援</h1>
                <p class="text-xs text-slate-400">V42 直覺化升級版 · 願每一次出勤，都有回家的路</p>
            </div>
        </div>
        <div class="flex items-center space-x-2">
            <button onclick="toggleModal('memorial-modal')" class="bg-slate-800 hover:bg-slate-700 px-3 py-1.5 rounded-lg text-xs font-medium border border-slate-700 transition"><i class="fa-solid fa-shield-heart mr-1 text-red-400"></i> 35位英雄致敬</button>
            <button onclick="toggleModal('archive-modal')" class="bg-red-600 hover:bg-red-500 px-3 py-1.5 rounded-lg text-xs font-medium transition shadow-md"><i class="fa-solid fa-book-open mr-1"></i> 300篇歷史索引</button>
        </div>
    </header>

    <!-- 頂部動態戰術安全提醒 (Toast) -->
    <div id="safety-toast" class="absolute top-20 left-1/2 transform -translate-x-1/2 z-20 max-w-lg w-11/12 glass-panel p-3 rounded-xl border-l-4 border-red-500 shadow-2xl transition-all duration-300">
        <div class="flex items-start justify-between">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-triangle-exclamation text-amber-400 text-lg"></i>
                <div>
                    <h4 id="toast-title" class="text-xs font-bold text-red-400">住宅用電與火場防護</h4>
                    <p id="toast-desc" class="text-xs text-slate-300">進入火場前務必確實檢查個人防護裝備 (PPE) 與空氣呼吸器殘壓。</p>
                </div>
            </div>
        </div>
    </div>

    <!-- 底部直覺化 Dock 控制列 (解決原先選單過多的問題) -->
    <nav class="absolute bottom-6 left-1/2 transform -translate-x-1/2 z-20 glass-panel p-2 rounded-2xl shadow-2xl flex items-center space-x-2 md:space-x-4">
        <button onclick="switchScene('residential')" class="dock-btn flex flex-col items-center px-3 py-2 rounded-xl bg-red-600 text-white transition transform hover:scale-105" data-scene="residential">
            <i class="fa-solid fa-house-chimney-crack text-lg"></i>
            <span class="text-[10px] mt-1">住宅用電</span>
        </button>
        <button onclick="switchScene('mountain')" class="dock-btn flex flex-col items-center px-3 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 transition transform hover:scale-105" data-scene="mountain">
            <i class="fa-solid fa-mountain text-lg"></i>
            <span class="text-[10px] mt-1">山域吊掛</span>
        </button>
        <button onclick="switchScene('water')" class="dock-btn flex flex-col items-center px-3 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 transition transform hover:scale-105" data-scene="water">
            <i class="fa-solid fa-water text-lg"></i>
            <span class="text-[10px] mt-1">水域救援</span>
        </button>
        <button onclick="switchScene('traffic')" class="dock-btn flex flex-col items-center px-3 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 transition transform hover:scale-105" data-scene="traffic">
            <i class="fa-solid fa-car-burst text-lg"></i>
            <span class="text-[10px] mt-1">交通救助</span>
        </button>
        <button onclick="switchScene('ems')" class="dock-btn flex flex-col items-center px-3 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 transition transform hover:scale-105" data-scene="ems">
            <i class="fa-solid fa-suitcase-medical text-lg"></i>
            <span class="text-[10px] mt-1">急診交接</span>
        </button>
    </nav>

    <!-- 35位英雄致敬彈窗 Modal -->
    <div id="memorial-modal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 backdrop-blur-sm hidden">
        <div class="glass-panel w-11/12 max-w-2xl max-h-[80vh] rounded-2xl p-6 flex flex-col shadow-2xl relative">
            <div class="flex justify-between items-center mb-4 border-b border-slate-700 pb-3">
                <h3 class="text-lg font-bold text-red-400"><i class="fa-solid fa-shield-heart mr-2"></i>35 位致敬人物與故事</h3>
                <button onclick="toggleModal('memorial-modal')" class="text-slate-400 hover:text-white text-lg"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="overflow-y-auto space-y-3 pr-2 text-xs">
                <div class="p-3 bg-slate-800/60 rounded-xl border border-slate-700">
                    <span class="text-red-400 font-bold">#01 導覽指揮官與基層守護者</span>
                    <p class="text-slate-300 mt-1">「每一次出勤，都有回家的路。」強調現場風險評估、適時撤退與資訊透明的重要性。</p>
                </div>
                <div class="p-3 bg-slate-800/60 rounded-xl border border-slate-700">
                    <span class="text-red-400 font-bold">#02 火場情境實務與指揮任務</span>
                    <p class="text-slate-300 mt-1">探討火場指揮官在動態高風險環境下的決策機制與安全管理。</p>
                </div>
                <div class="p-3 bg-slate-800/60 rounded-xl border border-slate-700">
                    <span class="text-red-400 font-bold">#03 裝備自主維護與防護升級</span>
                    <p class="text-slate-300 mt-1">致敬所有在日常扎實訓練、注重個人裝備及防護細節的消防同仁。</p>
                </div>
            </div>
        </div>
    </div>

    <!-- 300篇歷史文章索引彈窗 Modal -->
    <div id="archive-modal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 backdrop-blur-sm hidden">
        <div class="glass-panel w-11/12 max-w-3xl max-h-[80vh] rounded-2xl p-6 flex flex-col shadow-2xl relative">
            <div class="flex justify-between items-center mb-4 border-b border-slate-700 pb-3">
                <h3 class="text-lg font-bold text-red-400"><i class="fa-solid fa-book-open mr-2"></i>300 篇歷史圖文與議題索引</h3>
                <button onclick="toggleModal('archive-modal')" class="text-slate-400 hover:text-white text-lg"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="mb-3">
                <input type="text" placeholder="輸入關鍵字搜尋（例如：火災學、消防法、裝備、撤退）..." class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-2 text-xs text-white focus:outline-none focus:border-red-500">
            </div>
            <div class="overflow-y-auto space-y-2 pr-2 text-xs flex-grow">
                <div class="p-3 bg-slate-800/40 rounded-xl border border-slate-700/50 hover:border-red-500/50 transition">
                    <div class="flex justify-between text-slate-400 text-[10px] mb-1">
                        <span>日期：2026-10-12</span>
                        <span class="text-red-400">#火災學與防護</span>
                    </div>
                    <p class="font-medium text-slate-200">火場指揮官的任務與危險預判：在分秒必爭中確保同仁安全撤退。</p>
                </div>
                <div class="p-3 bg-slate-800/40 rounded-xl border border-slate-700/50 hover:border-red-500/50 transition">
                    <div class="flex justify-between text-slate-400 text-[10px] mb-1">
                        <span>日期：2026-09-28</span>
                        <span class="text-red-400">#消防法規與實務</span>
                    </div>
                    <p class="font-medium text-slate-200">從法規面檢視消防人員執勤安全保障與裝備升級需求。</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Three.js 3D 互動場景核心腳本 -->
    <script>
        // 場景資料與安全提醒設定
        const sceneData = {
            residential: { title: "住宅與用電安全救援", desc: "進入火場前務必檢查個人防護裝備 (PPE) 與空氣呼吸器殘壓。", color: 0xef4444, targetPos: { x: 0, y: 5, z: 15 } },
            mountain: { title: "山域繩索吊掛後送", desc: "高山救援需隨時評估天候與通訊死角，確保繩索系統雙重確保。", color: 0x10b981, targetPos: { x: 10, y: 6, z: 10 } },
            water: { title: "水域橡皮艇救援", desc: "湍流及水域作業務必穿戴合格救生衣與拋繩袋，注意上游水情。", color: 0x3b82f6, targetPos: { x: -10, y: 4, z: 12 } },
            traffic: { title: "車禍救助與破拆作業", desc: "落實車禍現場警戒、車體穩定支撐 (Cribbing) 與二次災害防範。", color: 0xf59e0b, targetPos: { x: 5, y: 3, z: 15 } },
            ems: { title: "急診交接與大量傷病患", desc: "落實生命徵象評估、TOCC 問診與檢傷分類，確保後送流暢。", color: 0x8b5cf6, targetPos: { x: -5, y: 3, z: 15 } }
        };

        // 初始化 Three.js
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x090d16);
        scene.fog = new THREE.FogExp2(0x090d16, 0.015);

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 12, 22);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        container.appendChild(renderer.domElement);

        // 燈光設定
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 1.2);
        dirLight.position.set(20, 40, 20);
        scene.add(dirLight);

        // 建立島嶼群 (低多邊形美學)
        const islandGroup = new THREE.Group();
        scene.add(islandGroup);

        // 主島
        const mainIslandGeo = new THREE.CylinderGeometry(8, 6, 3, 6);
        const mainIslandMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.8 });
        const mainIsland = new THREE.Mesh(mainIslandGeo, mainIslandMat);
        mainIsland.position.y = -1.5;
        islandGroup.add(mainIsland);

        // 中央燈塔
        const lighthouseGeo = new THREE.ConeGeometry(1.2, 8, 6);
        const lighthouseMat = new THREE.MeshStandardMaterial({ color: 0xd97706, roughness: 0.5 });
        const lighthouse = new THREE.Mesh(lighthouseGeo, lighthouseMat);
        lighthouse.position.set(0, 4, 0);
        islandGroup.add(lighthouse);

        // 動態海面
        const waterGeo = new THREE.PlaneGeometry(200, 200, 32, 32);
        const waterMat = new THREE.MeshStandardMaterial({ color: 0x1d4ed8, roughness: 0.2, metalness: 0.8, wireframe: false });
        const water = new THREE.Mesh(waterGeo, waterMat);
        water.rotation.x = -Math.PI / 2;
        water.position.y = -3;
        scene.add(water);

        // 互動狀態變數
        let currentTargetPos = { x: 0, y: 12, z: 22 };
        let isDragging = false;
        let previousMousePosition = { x: 0, y: 0 };

        // 滑鼠／觸控平移與旋轉控制（更直覺的互動）
        window.addEventListener('mousedown', (e) => { isDragging = true; previousMousePosition = { x: e.clientX, y: e.clientY }; });
        window.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            const deltaX = e.clientX - previousMousePosition.x;
            islandGroup.rotation.y += deltaX * 0.005;
            previousMousePosition = { x: e.clientX, y: e.clientY };
        });
        window.addEventListener('mouseup', () => { isDragging = false; });

        // 手機觸控支援
        window.addEventListener('touchstart', (e) => { if(e.touches.length === 1) { isDragging = true; previousMousePosition = { x: e.touches[0].clientX, y: e.touches[0].clientY }; } });
        window.addEventListener('touchmove', (e) => {
            if (!isDragging || e.touches.length !== 1) return;
            const deltaX = e.touches[0].clientX - previousMousePosition.x;
            islandGroup.rotation.y += deltaX * 0.005;
            previousMousePosition = { x: e.touches[0].clientX, y: e.touches[0].clientY };
        });
        window.addEventListener('touchend', () => { isDragging = false; });

        // 切換場景函數 (對應底部 Dock 按鈕)
        function switchScene(sceneKey) {
            // 更新按鈕樣式
            document.querySelectorAll('.dock-btn').forEach(btn => {
                btn.classList.remove('bg-red-600', 'text-white');
                btn.classList.add('bg-slate-800', 'text-slate-300');
            });
            const activeBtn = document.querySelector(`[data-scene="${sceneKey}"]`);
            if(activeBtn) {
                activeBtn.classList.remove('bg-slate-800', 'text-slate-300');
                activeBtn.classList.add('bg-red-600', 'text-white');
            }

            // 更新提示 Toast 內容
            const data = sceneData[sceneKey];
            document.getElementById('toast-title').innerText = data.title;
            document.getElementById('toast-desc').innerText = data.desc;

            // 平滑移動相機目標位置
            currentTargetPos = data.targetPos;
        }

        // 彈窗控制
        function toggleModal(modalId) {
            const modal = document.getElementById(modalId);
            modal.classList.toggle('hidden');
        }

        // 視窗縮放適應
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // 動畫循環
        let clock = new THREE.Clock();
        function animate() {
            requestAnimationFrame(animate);
            const elapsedTime = clock.getElapsedTime();

            // 海浪微微起伏動畫
            water.position.y = -3 + Math.sin(elapsedTime * 2) * 0.1;
            
            // 燈塔光束旋轉
            lighthouse.rotation.y = elapsedTime * 0.5;

            // 相機平滑滑動 (Lerp)
            camera.position.x += (currentTargetPos.x - camera.position.x) * 0.05;
            camera.position.y += (currentTargetPos.y - camera.position.y) * 0.05;
            camera.position.z += (currentTargetPos.z - camera.position.z) * 0.05;
            camera.lookAt(0, 0, 0);

            renderer.render(scene, camera);
        }
        animate();
    </script>
</body>
</html>
