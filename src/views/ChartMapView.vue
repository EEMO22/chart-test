<template>
    <div class="chart-container">
        <h2>지역별 시청 비율</h2>
        <canvas ref="mapCanvas"></canvas>
        <ul class="legend">
            <li v-for="(item, index) in countriesData" :key="index">
                {{ item.label }} <span>{{ item.value }}%</span>
            </li>
        </ul>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { Chart, registerables } from 'chart.js';
import { feature } from 'topojson-client';
import { ChoroplethController, GeoFeature, ProjectionScale, ColorScale } from 'chartjs-chart-geo';

Chart.register(...registerables, ChoroplethController, GeoFeature, ProjectionScale, ColorScale);

const mapCanvas = ref(null);
let chart;

//  국가 코드, 아이디 등은 iyuno data와 데이터 동기화 필요
const countriesData = [
    { label: '한국', value: 48, countryCode: 'KOR' },
    { label: '인도네시아', value: 20, countryCode: 'IDN' },
    { label: '일본', value: 10, countryCode: 'JPN' },
    { label: '영국', value: 10, countryCode: 'GBR' },
    { label: '캐나다', value: 8, countryCode: 'CAN' },
    { label: '기타', value: 4 }
];

const countryIdToCodeMap = {
    392: 'JPN',
    360: 'IDN',
    410: 'KOR',
    826: 'GBR',
    124: 'CAN',
};

onMounted(async () => {
    try {
        const response = await fetch('https://unpkg.com/world-atlas/countries-50m.json');
        const data = await response.json();
        const countries = feature(data, data.objects.countries).features.filter(d => d.properties.name !== 'Antarctica');

        const datasetData = countries.map(d => {
            const countryCode = countryIdToCodeMap[d.id];
            const country = countriesData.find(c => c.countryCode === countryCode);

            return {
                feature: d,
                value: country ? country.value : 0,
            };
        });

        chart = new Chart(mapCanvas.value.getContext('2d'), {
            type: 'choropleth',
            data: {
                labels: countries.map(d => d.properties.name),
                datasets: [{
                    label: 'Countries',
                    data: datasetData,
                }]
            },
            options: {
                showOutline: true,
                showGraticule: false,
                plugins: {
                    legend: {
                        display: false
                    },
                },
                scales: {
                    projection: {
                        axis: 'x',
                        projection: 'geoNaturalEarth1',
                        projectionScale: 1,
                    },
                    color: {
                        axis: 'x',
                        interpolate: (v) => {
                            if (v > 0.5) {
                                return '#3342AA';
                            } else if (v > 0.2) {
                                return '#3D85F1';
                            } else if (v > 0.1) {
                                return '#95C9FF';
                            } else {
                                return 'black';
                            }
                        },
                        display: false,
                    }
                }
            }
        });
    } catch (error) {
        console.error('Error loading or rendering the chart:', error);
    }
});

onBeforeUnmount(() => {
    if (chart) {
        chart.destroy();
    }
});
</script>

<style scoped>
.chart-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    /* background-color: #1c1c1e; */
    padding: 20px;
    border-radius: 8px;
    color: white;
    max-width: 800px;
    margin: 0 auto;
}

h2 {
    margin-bottom: 20px;
    font-size: 1.5em;
}

canvas {
    width: 100%;
    height: auto;
    max-width: 600px;
}

.legend {
    list-style: none;
    padding: 0;
    margin: 20px 0 0;
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.legend li {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.legend span {
    display: inline-block;
    width: 20px;
    height: 20px;
    margin-right: 10px;
    border-radius: 4px;
}
</style>
