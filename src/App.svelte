<script>
	import { json } from 'd3';
	import { geoPath, geoNaturalEarth1, scaleLinear, extent, hsl } from 'd3';
	import { createEventDispatcher } from 'svelte';
	import { raise } from 'layercake';
	import { draw } from 'svelte/transition';
	import { quadInOut } from 'Svelte/easing';

	let dataset = [];
	let dataset2 = [];

	const sphere = { type: 'Sphere' };
	var height = window.innerHeight * 0.95;
	var width = window.innerWidth * 0.95;

	let colorScale = () => {};

	const projection = geoNaturalEarth1().fitSize([width, height], sphere);

	const path = geoPath(projection);

	//fetch both json file
	json(
		'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	).then((data1) => {
		//data has all the information of countries: names, id, geomatrical data, etc.
		//then save all that into an array named "dataset"
		dataset = data1.features;
		json('https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.json').then((data2) => {
			for(let key in data2){
				dataset2.push(data2[key]);
			}
			//add energy data into the geojson file, under properties
			//some countries will be missing energy data due to the missing ID
			dataset.forEach((i) => {
				dataset2.forEach((j) => {
					if (i.id == j.iso_code){
						i.properties.data = j.data
					}
				})
			})
			console.log(dataset)
			const nameExtent = extent(dataset, d => d.properties.name.length);
			colorScale = scaleLinear().domain(nameExtent).range(["white", "black"])

		})

		
	});

	//svelte event dispatcher(?)
	const dispatch = createEventDispatcher();

	function handleMousemove(feature) {
		return function handleMousemoveFn(e) {
			//raise: Layercake component(?) Used to handle borders of neighboring countries
			raise(this);
			if (e.layerX !== 0 && e.layerY !== 0) {

				//could be named however 
				//props function unknown, seems to be used to access what is being passed in here from other svelte file
				dispatch('eventname', {e, props: feature })
			}
		}
	}

</script>

<main>
	<svg viewBox="0 0 {width} {height}">
		{#each dataset as data}
			<path
				class="feature-path"
				transition:draw={{ duration: 2000, delay: 0, easing: quadInOut }}
				d = {path(data)}
				on:mousemove={handleMousemove(data)}
				fill = {colorScale(data.properties.name.length)}
			/>
		{/each}
	</svg>
</main>

<style>
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	path {
		stroke: gray;
	}

	.feature-path:hover {
		stroke: rgb(255, 255, 255);
		stroke-width: 1.5px;
	}
</style>
