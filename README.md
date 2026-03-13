<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>기저 부하 안정성 실증 분석</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css">
    <style>
        body { font-family: 'Pretendard', sans-serif; background: #f8fafc; color: #1e293b; }
        .panel { background: #ffffff; border: 1px solid #e2e8f0; border-radius: 20px; box-shadow: 0 1px 3px rgba(0,0,0,0.02); }
        .label-sm { font-size: 0.75rem; font-weight: 800; color: #64748b; letter-spacing: 0.05em; }
        .value-main { font-size: 3.5rem; font-weight: 900; letter-spacing: -0.05em; line-height: 1; }
        input[type="range"] { -webkit-appearance: none; background: #f1f5f9; height: 6px; border-radius: 10px; width: 100%; cursor: pointer; }
        input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 16px; height: 16px; background: #2563eb; border-radius: 50%; border: 3px solid #fff; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .shield-layer { height: 10px; border-radius: 5px; background: #f1f5f9; overflow: hidden; }
        .shield-fill { height: 100%; background: #2563eb; transition: width 0.7s cubic-bezier(0.4, 0, 0.2, 1); }
    </style>
</head>
<body class="p-6 md:p-12">

    <div class="max-w-6xl mx-auto">
        <header class="mb-10 pb-6 border-b border-slate-200">
            <h1 class="text-2xl font-black text-slate-900 tracking-tight text-center md:text-left">국가 기저 부하 전력망 안정성 분석</h1>
            <p class="text-sm text-slate-500 mt-2 font-medium text-center md:text-left">데이터 시뮬레이션을 통한 원자력의 전력 계통 기여도 및 물리적 안전 계통 탐구</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <aside class="lg:col-span-4 space-y-6">
                <div class="panel p-8">
                    <h2 class="text-sm font-black mb-8 text-slate-400 border-b pb-2 uppercase tracking-tighter">Environment Control</h2>
                    <div class="mb-10">
                        <label class="label-sm block mb-4 text-blue-600">원자력 : 우라늄 연료 투입량 (g)</label>
                        <input type="number" id="u-mass" value="1" class="w-full text-blue-600 mb-2 bg-transparent font-black text-2xl outline-none" oninput="calculate()">
                        <p class="text-[10px] text-slate-400">※ 연료 부족 시 기저 부하 유지 능력이 저하됩니다.</p>
                    </div>
                    <div class="space-y-8 pt-6 border-t border-slate-50">
                        <div>
                            <label class="label-sm block mb-2 font-bold">화력 : 발전 효율 변수 (<span id="anthra-val" class="text-slate-900 font-black">20</span>%)</label>
                            <input type="range" id="anthra-ratio" min="0" max="100" value="20" oninput="updateRangeLabel('anthra')">
                        </div>
                        <div>
                            <div class="flex justify-between items-center mb-2">
                                <label class="label-sm block font-bold">풍력 : 현재 풍속 환경 (<span id="wind-val" class="text-slate-900 font-black">12</span> m/s)</label>
                                <span id="wind-status" class="text-[10px] font-black px-2 py-0.5 rounded bg-blue-50 text-blue-600">정상 가동</span>
                            </div>
                            <input type="range" id="wind-speed" min="0" max="40" value="12" oninput="updateRangeLabel('wind')">
                        </div>
                    </div>
                </div>

                <div class="panel p-8">
                    <h3 class="text-sm font-black text-slate-800 mb-6 flex items-center gap-2">물리적 5중 방호벽 가동 상태</h3>
                    <div class="space-y-4">
                        <div class="space-y-1">
                            <div class="flex justify-between text-[10px] font-bold text-slate-500"><span>1-2단: 연료 펠렛 및 피복재</span><span id="shield-val-1">100%</span></div>
                            <div class="shield-layer"><div id="shield-bar-1" class="shield-fill" style="width: 100%"></div></div>
                        </div>
                        <div class="space-y-1">
                            <div class="flex justify-between text-[10px] font-bold text-slate-500"><span>3단: 원자로 강철 압력 용기</span><span id="shield-val-2">100%</span></div>
                            <div class="shield-layer"><div id="shield-bar-2" class="shield-fill" style="width: 100%"></div></div>
                        </div>
                        <div class="space-y-1">
                            <div class="flex justify-between text-[10px] font-bold text-slate-500"><span>4-5단: 콘크리트 격납 건물</span><span id="shield-val-3">100%</span></div>
                            <div class="shield-layer"><div id="shield-bar-3" class="shield-fill" style="width: 100%"></div></div>
                        </div>
                    </div>
                </div>
            </aside>

            <div class="lg:col-span-8 space-y-6">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="panel p-8 border-l-4 border-l-blue-600">
                        <h4 class="label-sm mb-6">원자력 에너지 밀도 효율</h4>
                        <div class="flex items-baseline gap-1">
                            <span class="value-main text-blue-600" id="u-ratio-text">0</span>
                            <span class="text-sm font-bold text-slate-400">배</span>
                        </div>
                    </div>
                    <div class="panel p-8 border-l-4 border-l-slate-800">
                        <h4 class="label-sm mb-6">전력망 공급 안정성 (Grid Stability)</h4>
                        <div class="flex items-center gap-4">
                            <span class="value-main text-slate-800" id="stability-score">S</span>
                            <div class="flex-1 bg-slate-50 h-2 rounded-full overflow-hidden">
                                <div id="stability-bar" class="h-full transition-all duration-700 ease-out"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="panel p-10">
                    <h3 class="text-sm font-black text-slate-800 mb-8 border-b pb-4 uppercase tracking-widest">Energy Output Profile</h3>
                    <div class="h-[300px]"><canvas id="mainChart"></canvas></div>
                </div>

                <div class="panel p-8 bg-slate-50/50 border-dashed border-2 border-slate-200">
                    <h3 class="text-sm font-black text-slate-800 mb-6 italic">전문 탐구 : 기저 부하(Base-load)와 계통 안정성</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-8 text-[11px] leading-relaxed text-slate-600">
                        <div><p class="font-black text-slate-900 mb-2">■ 왜 기저 부하가 중요한가?</p>재생에너지의 불확실성을 상쇄하기 위해 원자력과 같은 고정 출력 전원이 전력망의 균형을 유지해야 블랙아웃을 방지할 수 있습니다.</div>
                        <div><p class="font-black text-slate-900 mb-2">■ 기술적 안정성의 결론</p>풍속 등 자연 조건에 민감한 풍력과 달리 원자력은 일정한 출력을 유지하여 산업 인프라의 신뢰도를 보장합니다.</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let myChart;
        function updateRangeLabel(type) {
            if(type === 'anthra') document.getElementById('anthra-val').innerText = document.getElementById('anthra-ratio').value;
            if(type === 'wind') document.getElementById('wind-val').innerText = document.getElementById('wind-speed').value;
            calculate();
        }

        function calculate() {
            const coalMass = 1000;
            const anthraRatio = document.getElementById('anthra-ratio').value / 100;
            const coalKwh = (coalMass * (6500 * (1 - anthraRatio) + 4500 * anthraRatio) / 860) * 0.38;

            const v = parseFloat(document.getElementById('wind-speed').value);
            const statusTag = document.getElementById('wind-status');
            let windKwh = 0.1;
            
            if (v < 3) {
                statusTag.innerText = "풍속 부족"; statusTag.className = "text-[10px] font-black px-2 py-0.5 rounded bg-slate-100 text-slate-400";
            } else if (v <= 25) {
                windKwh = (0.5 * 1.225 * (Math.PI * Math.pow(50, 2)) * Math.pow(v, 3) * 0.4) / 1000;
                statusTag.innerText = "정상 가동"; statusTag.className = "text-[10px] font-black px-2 py-0.5 rounded bg-blue-50 text-blue-600";
            } else if (v < 27) {
                windKwh = ((0.5 * 1.225 * (Math.PI * Math.pow(50, 2)) * Math.pow(25, 3) * 0.4) / 1000) * ((27 - v) / 2);
                statusTag.innerText = "안전 모드 진입"; statusTag.className = "text-[10px] font-black px-2 py-0.5 rounded bg-amber-50 text-amber-600";
            } else {
                statusTag.innerText = "완전 정지 (컷아웃)"; statusTag.className = "text-[10px] font-black px-2 py-0.5 rounded bg-rose-50 text-rose-600";
            }

            const uMass = parseFloat(document.getElementById('u-mass').value) || 0;
            const uKwh = uMass * 20000; 
            updateUI(coalKwh, windKwh, uKwh, v, uMass);
        }

        function updateUI(c, w, u, currentWind, uMass) {
            document.getElementById('u-ratio-text').innerText = (u / c).toFixed(1);
            
            // 버그 수정: 전력망 안정성 점수 계산 (실시간 반영)
            let baseStability = 100;
            if (currentWind > 25) baseStability -= (currentWind - 25) * 45;
            else if (currentWind < 3) baseStability -= 20;
            if (uMass < 0.5) baseStability -= (0.5 - uMass) * 100; // 원자력 연료가 적으면 안정성 급락

            baseStability = Math.max(5, Math.min(100, baseStability));
            const bar = document.getElementById('stability-bar');
            const score = document.getElementById('stability-score');
            
            bar.style.width = baseStability + "%";
            bar.style.backgroundColor = baseStability > 70 ? "#2563eb" : "#f43f5e";
            score.innerText = baseStability > 85 ? "S" : (baseStability > 50 ? "A" : "B");

            // 버그 수정: 5중 방호벽 상태 업데이트 (원자력 가동 상태와 연동)
            const shieldFactor = uMass > 0 ? 100 : 0; // 연료가 아예 없으면 시스템 비활성화 시각화
            for(let i=1; i<=3; i++) {
                document.getElementById(`shield-bar-${i}`).style.width = shieldFactor + "%";
                document.getElementById(`shield-val-${i}`).innerText = shieldFactor + "%";
            }

            updateChart(c, w, u);
        }

        function updateChart(c, w, u) {
            const ctx = document.getElementById('mainChart').getContext('2d');
            if (!myChart) {
                myChart = new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: ['화력 (석탄)', '풍력 (자연)', '원자력 (우라늄)'],
                        datasets: [{ data: [c, w, u], backgroundColor: ['#f1f5f9', '#94a3b8', '#2563eb'], borderRadius: 8, barPercentage: 0.35 }]
                    },
                    options: {
                        responsive: true, maintainAspectRatio: false,
                        animation: { duration: 800, easing: 'easeOutQuart' },
                        plugins: { legend: { display: false } },
                        scales: { y: { type: 'logarithmic', grid: { display: false }, ticks: { display: false } }, x: { grid: { display: false }, ticks: { font: { weight: '800' }, color: '#475569' } } }
                    }
                });
            } else {
                myChart.data.datasets[0].data = [c, w, u];
                myChart.update(); 
            }
        }
        calculate();
    </script>
</body>
</html>
