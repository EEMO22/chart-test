<template>
    <div class="container">
        <div style="height: 300px;">
            <canvas ref="lineChartCanvas" width="800" height="300"></canvas>
        </div>
        <div style="height: 200px; margin-top: 20px;">
            <canvas ref="barChartCanvas" width="800" height="200"></canvas>
        </div>
        <div id="custom-tooltip"></div>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { Chart, registerables } from 'chart.js';

const labels = [
    '8월 1일', '8월 2일', '8월 3일', '8월 4일', '8월 5일', '8월 6일', '8월 7일',
    '8월 8일', '8월 9일', '8월 10일', '8월 11일', '8월 12일', '8월 13일', '8월 14일',
    '8월 15일', '8월 16일', '8월 17일', '8월 18일', '8월 19일', '8월 20일', '8월 21일',
    '8월 22일', '8월 23일', '8월 24일', '8월 25일', '8월 26일', '8월 27일', '8월 28일',
    '8월 29일', '8월 30일', '8월 31일'
];
const viewCounts = [2000, 2200, 2400, 3000, 3400, 3800, 5000, 5500, 6000, 7000, 8000, 8500, 10000, 11000, 11500, 15000, 16000, 18000, 20000, 21000, 21500, 22145, 22500, 22700, 23000, 23500, 24000, 25000, 26000, 26500, 27000];  // 누적 조회수 데이터
const dailyIncreases = [100, 150, 200, 250, 300, 350, 400, 450, 500, 550, 600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050, 1100, 1150, 1200, 1250, 1300, 1350, 1400, 1450, 1500, 1550, 1600];  // 일일 증가량 데이터

Chart.register(...registerables);

const lineChartCanvas = ref(null);
const barChartCanvas = ref(null);
let lineChart, barChart;

const handleMouseLeave = () => {
    const tooltipEl = document.getElementById('custom-tooltip');
    if (tooltipEl) {
        tooltipEl.style.opacity = 0;
    }
    if (lineChart) {
        lineChart.setActiveElements([]);
        lineChart.update('none');
    }
    if (barChart) {
        barChart.setActiveElements([]);
        barChart.update('none');
    }
};

const crosshairPlugin = {
    id: 'crosshair',
    defaults: {
        width: 1,
        color: 'rgba(255, 255, 255, 0.5)',
        dashPattern: [5, 5],
    },
    afterInit: (chart) => {
        chart.crosshair = {
            x: 0,
            y: 0,
        };
    },
    afterEvent: (chart, args) => {
        const { inChartArea } = args;
        const { x, y } = args.event;

        if (inChartArea) {
            const xAxis = chart.scales.x;
            const nearestIndex = xAxis.getValueForPixel(x);
            const snappedX = xAxis.getPixelForValue(nearestIndex);
            chart.crosshair = { x: snappedX, y };
            syncCrosshair(chart);
        } else {
            chart.crosshair = { x: 0, y: 0 };
            syncCrosshair(chart);
        }
    },
    afterDatasetsDraw: (chart, args, options) => {
        const { ctx, chartArea: { top, bottom }, crosshair } = chart;
        if (crosshair.x > 0) {
            ctx.save();
            ctx.beginPath();
            ctx.moveTo(crosshair.x, top);
            ctx.lineTo(crosshair.x, bottom);
            ctx.lineWidth = options.width || 1;
            ctx.strokeStyle = options.color || 'rgba(255,255,255,0.5)';
            ctx.stroke();
            ctx.restore();
        }
    },
};

Chart.register(crosshairPlugin);

const syncCrosshair = (sourceChart) => {
    const targetChart = sourceChart === lineChart ? barChart : lineChart;

    console.log('targetChart', targetChart.data.datasets);

    if (targetChart && targetChart.ctx) {
        targetChart.crosshair = sourceChart.crosshair;
        targetChart.options.plugins.crosshair.x = sourceChart.crosshair.x;
        targetChart.update('none');
        targetChart.draw();
    }
};

const syncTooltip = (context) => {
    const tooltipEl = document.getElementById('custom-tooltip');
    const { chart, tooltip } = context;

    if (tooltip.opacity === 0) {
        tooltipEl.style.opacity = 0;
        return;
    }

    const index = tooltip.dataPoints[0].dataIndex;
    const date = labels[index];
    const viewCount = viewCounts[index];
    const dailyIncrease = dailyIncreases[index];

    tooltipEl.innerHTML = `
    <div style="background: rgba(0,0,0,0.7); color: white; padding: 8px; border-radius: 4px;">
      <div>${date}</div>
      <div>누적 조회수: ${viewCount.toLocaleString()}회</div>
      <div>일일 증가량: ${dailyIncrease.toLocaleString()}회</div>
    </div>
  `;

    const { offsetLeft: canvasLeft, offsetTop: canvasTop } = chart.canvas;

    const tooltipWidth = tooltipEl.offsetWidth;
    const tooltipHeight = tooltipEl.offsetHeight;

    const xPosition = canvasLeft + tooltip.caretX - tooltipWidth / 2;

    const chartHeightTotal = document.getElementsByClassName('container')[0].clientHeight;
    const yPosition = chartHeightTotal / 2;

    tooltipEl.style.left = `${xPosition}px`;
    tooltipEl.style.top = `${yPosition}px`;
    tooltipEl.style.opacity = 1;
};

const createLineChart = () => {
    if (!lineChartCanvas.value) return;

    const ctx = lineChartCanvas.value.getContext('2d');
    const gradient = ctx.createLinearGradient(0, 0, 0, 400);
    gradient.addColorStop(0, 'rgba(75, 192, 192, 0.4)');
    gradient.addColorStop(1, 'rgba(75, 192, 192, 0)');

    lineChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: '누적 조회수',
                data: viewCounts,
                borderColor: 'rgba(75, 192, 192, 1)',
                borderWidth: 2,
                backgroundColor: gradient,
                fill: true,
                pointRadius: 0,
                pointHoverRadius: 7,
                tension: 0.3
            }]
        },
        options: {
            scales: {
                x: {
                    type: 'category',
                    grid: { display: false },
                    ticks: { maxTicksLimit: 10 }
                },
                y: {
                    beginAtZero: true,
                    display: false,
                    ticks: {
                        color: 'rgba(255, 255, 255, 0.7)',
                        callback: value => `${value.toLocaleString()}회`
                    }
                }
            },
            plugins: {
                tooltip: { enabled: false },
                crosshair: {
                    color: 'rgba(255, 255, 255, 0.5)',
                    width: 1
                },
                legend: { display: false },
            },
            responsive: true,
            maintainAspectRatio: false,
            interaction: {
                intersect: false,
                mode: 'index',
            },
            onHover: (event, activeElements, chart) => {
                if (activeElements.length > 0) {
                    const index = activeElements[0].index;
                    barChart.setActiveElements([{ datasetIndex: 0, index }]);
                    barChart.update('none');

                    syncTooltip({
                        chart: chart,
                        tooltip: {
                            opacity: 1,
                            dataPoints: [{ dataIndex: index }],
                            caretX: chart.scales.x.getPixelForValue(index)
                        }
                    });
                } else {
                    document.getElementById('custom-tooltip').style.opacity = 0;
                }
            },
        }
    });
};

const createBarChart = () => {
    if (!barChartCanvas.value) return;

    const ctx = barChartCanvas.value.getContext('2d');

    barChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: labels,
            datasets: [{
                label: '일일 증가량',
                data: dailyIncreases,
                backgroundColor: 'rgba(54, 162, 235, 0.5)',
                borderWidth: 1
            }]
        },
        options: {
            scales: {
                x: {
                    display: false,
                    barPercentage: 0.5,
                },
                y: { display: false }
            },
            plugins: {
                tooltip: { enabled: false },
                crosshair: {
                    color: 'rgba(255, 255, 255, 0.5)',
                    width: 1
                },
                legend: { display: false },
            },
            responsive: true,
            maintainAspectRatio: false,
            interaction: {
                intersect: false,
                mode: 'index',
            },
            onHover: (event, activeElements, chart) => {
                if (activeElements.length > 0) {
                    const index = activeElements[0].index;
                    lineChart.setActiveElements([{ datasetIndex: 0, index }]);
                    lineChart.update('none');

                    syncTooltip({
                        chart: chart,
                        tooltip: {
                            opacity: 1,
                            dataPoints: [{ dataIndex: index }],
                            caretX: chart.scales.x.getPixelForValue(index)
                        }
                    });
                } else {
                    document.getElementById('custom-tooltip').style.opacity = 0;
                }
            },
        }
    });
};

onMounted(() => {
    console.log('mounted');
    createLineChart();
    createBarChart();

    if (lineChartCanvas.value) {
        lineChartCanvas.value.addEventListener('mouseleave', handleMouseLeave);
    }

    if (barChartCanvas.value) {
        barChartCanvas.value.addEventListener('mouseleave', handleMouseLeave);
    }
});

onBeforeUnmount(() => {
    console.log('before unmount');
    if (lineChart) {
        lineChart.destroy();
    }

    if (barChart) {
        barChart.destroy();
    }

    if (lineChartCanvas.value) {
        lineChartCanvas.value.removeEventListener('mouseleave', handleMouseLeave);
    }

    if (barChartCanvas.value) {
        barChartCanvas.value.removeEventListener('mouseleave', handleMouseLeave);
    }
});
</script>

<style lang="scss" scoped>
.container {
    position: relative;

    #custom-tooltip {
        position: absolute;
        top: 0;
        min-width: 160px;
        pointer-events: none;
        opacity: 0;
    }
}
</style>