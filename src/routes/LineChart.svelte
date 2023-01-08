<script>
	import Chart from 'chart.js/auto';
	import { onMount } from 'svelte';

	export let points;
	export let points2;
	export let yAxisLabel;
	export let xAxisLabel;
	let generateLabels = function (deltaVal, numberOfPoints) {
		let tempArray = [];
		for (let i = 0; i < numberOfPoints; i++) {
			tempArray.push(`${i * deltaVal}`);
		}
		return tempArray;
	};

	let ctx;
	let chart;

	$: if (chart) {
		chart.data.datasets[0].data = points;
		chart.data.datasets[1].data = points2;
		chart.data.labels = generateLabels(1, points.length);
		chart.update();
	}

	onMount(async () => {
		chart = new Chart(ctx, {
			type: 'line',
			options: {
				plugins: {
					legend: {
						display: false
					},
					tooltip: {
						enabled: false
					}
				},
				scales: {
					y: {
						title: {
							display: true,
							text: yAxisLabel,
							font: {
								family: 'monospace',
								style: 'italic'
							}
						}
					},
					x: {
						title: {
							display: true,
							text: xAxisLabel,
							font: {
								family: 'monospace',
								style: 'italic'
							}
						}
					}
				}
			},
			data: {
				labels: generateLabels(1, points.length),
				datasets: [
					{
						label: 'Visualized Graph',
						data: points,
						lineTension: 0.35,
						fill: true
					},
					{
						label: 'Visualized Graph',
						data: points2,
						lineTension: 0.35,
						fill: true
					}
				]
			}
		});
	});
</script>

<div id="chart-container">
	<canvas id="chart" width="500" height="300" bind:this={ctx} />
</div>

<!-- Man i really dislike the labeling system of chart.js-->
