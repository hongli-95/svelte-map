<script>
	import { json } from 'd3';
	import Marks from './Marks.svelte';

	let dataset = [];
	json(
		'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	).then((data) => {
		dataset = data.features;
	});

	const width = window.innerWidth;
	const height = window.innerHeight;
</script>

<main>
	<svg viewBox="0 0 {width} {height}">
		<Marks {dataset} />
	</svg>
</main>

<style>
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
