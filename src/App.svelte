<script>
	import { json } from 'd3';
	import Marks from './Marks.svelte';

	let dataset = [];
	//let idList = [];

	json(
		'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	).then((data) => {
		//data has all the information of countries: names, id, geomatrical data, etc.
		//then save all that into an array named "dataset"
		dataset = data.features;

		// for (let i = 0; i < dataset.length; i++) {
		// 	idList.push(dataset[i].id)
		// }
		// console.log(idList);
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
