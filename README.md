<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>에너지 효율 분석 시스템</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css">
    <style>
        body { font-family: 'Pretendard', sans-serif; background: #fdfdfd; color: #1e293b; }
        .panel { background: #ffffff; border: 1px solid #e5e7eb; border-radius: 16px; box-shadow: 0 4px 20px -5px rgba(0,0,0,0.03); }
        .accent-blue { border-left: 4px solid #2563eb; }
        .label-sm { font-size: 0.75rem; font-weight: 700; color: #64748b; letter-spacing: -0.01em; }
        .value-main { font-size: 3.5rem; font-weight: 900; letter-spacing: -0.05em; line-height: 1; }
        
        /* 입력창 및 슬라이더 정제 */
        input[type="number"] { border-bottom: 2px solid #e2e8f0; padding: 4px 0; font-weight: 800; font-size: 1.25rem; outline: none; transition: border-color 0.2s; }
        input[type="number"]:focus { border-color: #2563eb; }
        input[type="range"] { -webkit-appearance: none; background: #f1f5f9; height: 4px; border-radius: 10px; width: 100%; }
        input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 14px; height: 14px; background: #2563eb; border-radius: 50%; cursor: pointer; }
    </style>
</head>
<body class="p-6 md:p-12">

    <div class="max-w-6xl mx-auto">
        <header class="mb-10 border-b border-gray-100 pb-6">
            <h1 class="text-2xl font-black text-slate-800 tracking-tight">원자력 에너지 효율 및 탄소 저감 시뮬레이션</h1>
            <p class="text-sm text-slate-400 mt-1 font-medium italic">차세대 에너지 믹스 구성을 위한 공학적 비교 분석</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            
            <aside class="lg:col-span-4 space-y-6">
                <div class="panel p-8">
                    <h2 class="text-sm font-black mb-8 text-slate-500 uppercase tracking-widest border-b pb-2">실험 매개변수 설정</h2>
                    
                    <div class="mb-10">
                        <label class="label-sm block mb-4 text-blue-600">원자력 발전 : 우라늄 투입량 (g)</label>
                        <input type="number" id="u-mass" value="1" class="w-full text-blue-600 mb-2">
                        <p class="text-[10px] text-slate-400 leading-normal">우라늄 1g은 약 20,000kWh의 전력을 생산하며, 이는 석탄 수 톤에 해당하는 에너지 밀도를 가집니다.</p>
                    </div>

                    <div class="space-y-8 pt-6 border-t border-gray-50">
                        <div>
                            <label class="label-sm block mb-2">화력 발전 : 무연탄 혼합 비중 (<span id="anthra-val" class="text-slate-800 font-bold">20</span>%)</label>
                            <input type="range" id="anthra-ratio" min="0" max="100" value="20">
                        </div>

                        <div>
                            <label class="label-sm block mb-2">풍력 발전 : 현재 풍속 환경 (<span id="wind-val" class="text-slate-800 font-bold">12</span> m/s)</label>
                            <input type="range" id="wind-speed" min="0" max="35" value="12">
                        </div>
                    </div>

                    <button onclick="calculate()" class="w-full mt-10 py-4 bg-slate-900 text-white font-bold rounded-lg hover:bg-black transition-all shadow-lg active:scale-95">
                        데이터 분석 실행
                    </button>
                </div>
            </aside>

            <div class="lg:col-span-8 space-y-6">
                
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="panel p-8 accent-blue">
                        <h4 class="label-sm mb-6">에너지 지배력 지수</h4>
                        <div class="flex items-baseline gap-2">
                            <span class="value-main text-blue-600" id="u-ratio-text">0</span>
                            <span class="text-sm font-bold text-slate-400">배</span>
                        </div>
                        <div class="mt-6 text-[11px] text-slate-500 border-t pt-4">
                            <span class="font-bold text-slate-700 block mb-1">[산정 기준] 에너지 밀도 (Energy Density)</span>
                            우라늄 1g이 생산하는 에너지를 화력 발전이 따라잡기 위해 투입해야 할 석탄의 질량 배율입니다.
                        </div>
                    </div>

                    <div class="panel p-8 border-l-4 border-l-slate-800">
                        <h4 class="label-sm mb-6 text-slate-400">탄소 저감 등급</h4>
                        <div class="flex items-center gap-4">
                            <span class="value-main text-slate-800" id="carbon-score">S</span>
                            <div class="flex-1 bg-slate-100 h-2 rounded-full overflow-hidden">
                                <div id="carbon-bar" class="h-full bg-blue-600 transition-all duration-1000"></div>
                            </div>
                        </div>
                        <div class="mt-6 text-[11px] text-slate-500 border-t pt-4">
                            <span class="font-bold text-slate-700 block mb-1">[산정 기준] 전 생애 주기 평가 (LCA)</span>
                            무탄소 전원인 원자력과 풍력의 합산 발전 비중을 통해 환경적 가치를 정량화한 지수입니다.
                        </div>
                    </div>
                </div>

                <div class="panel p-8">
                    <h3 class="text-sm font-black text-slate-800 mb-8 border-b pb-2">에너지원별 실시간 출력 프로필 (kWh)</h3>
                    <div class="h-[320px]">
                        <canvas id="mainChart"></canvas>
                    </div>
                    <div class="mt-6 flex justify-end gap-6 text-[10px] font-bold text-slate-400">
                        <div class="flex items-center gap-1.5"><span class="w-2 h-2 bg-slate-200 rounded-full"></span> 화력(석탄)</div>
                        <div class="flex items-center gap-1.5"><span class="w-2 h-2 bg-slate-400 rounded-full"></span> 풍력(자연)</div>
                        <div class="flex items-center gap-1.5"><span class="w-2 h-2 bg-blue-600 rounded-full"></span> 원자력(우라늄)</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let myChart;
        
        document.getElementById('anthra-ratio').oninput = function() { document.getElementById('anthra-val').innerText = this.value; calculate(); }
        document.getElementById('wind-speed').oninput = function() { document.getElementById('wind-val').innerText = this.value; calculate(); }

        function calculate() {
            // 비교 기준: 석탄 1,000kg (1톤)
            const coalMass = 1000;
            const anthraRatio = document.getElementById('anthra-ratio').value / 100;
            const mixedHeat = 6500 * (1 - anthraRatio) + 4500 * anthraRatio;
            const coalKwh = (coalMass * mixedHeat / 860) * 0.38;

            // 풍력 계산 (컷아웃 로직)
            const v = parseFloat(document.getElementById('wind-speed').value);
            let windKwh = (v >= 3 && v <= 25) ? (0.5 * 1.225 * (Math.PI * Math.pow(50, 2)) * Math.pow(v, 3) * 0.4) / 1000 : 0.1;

            // 원자력 계산 (메인 변수)
            const uMass = parseFloat(document.getElementById('u-mass').value) || 0;
            const uKwh = uMass * 20000; 

            updateUI(coalKwh, windKwh, uKwh);
        }

        function updateUI(c, w, u) {
            const uRatio = (u / c).toFixed(1);
            document.getElementById('u-ratio-text').innerText = parseFloat(uRatio).toLocaleString();
            
            const cleanRatio = ((w + u) / (c + w + u || 1)) * 100;
            const bar = document.getElementById('carbon-bar');
            const score = document.getElementById('carbon-score');
            
            bar.style.width = cleanRatio + "%";
            score.innerText = cleanRatio > 90 ? "S+" : (cleanRatio > 70 ? "S" : "A");
            
            updateChart(c, w, u);
        }

        function updateChart(c, w, u) {
            const ctx = document.getElementById('mainChart').getContext('2d');
            if (myChart) myChart.destroy();
            
            myChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['화력 발전', '풍력 발전', '원자력 발전'],
                    datasets: [{
                        data: [c, w, u],
                        backgroundColor: ['#e2e8f0', '#94a3b8', '#2563eb'],
                        borderRadius: 8,
                        barPercentage: 0.4
                    }]
                },
                options: {
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: {
                        y: { 
                            type: 'logarithmic', 
                            grid: { color: '#f8fafc', drawBorder: false },
                            ticks: { font: { size: 10, family: 'Pretendard' }, color: '#94a3b8' }
                        },
                        x: { 
                            grid: { display: false },
                            ticks: { font: { weight: 'bold', family: 'Pretendard' }, color: '#475569' }
                        }
                    }
                }
            });
        }

        calculate();
    </script>
</body>
</html>
