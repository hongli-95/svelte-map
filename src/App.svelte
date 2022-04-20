<script>
	import { json, csv } from 'd3';
	import { geoPath, geoNaturalEarth1, scaleLinear, extent } from 'd3';
	import { createEventDispatcher } from 'svelte';
	import { raise } from 'layercake';
	import { draw } from 'svelte/transition';
	import { quadInOut } from 'Svelte/easing';

	let dataset = [];

	const sphere = { type: 'Sphere' };
	var height = window.innerHeight * 0.95;
	var width = window.innerWidth * 0.95;

	//color scale function for scaleLinear
	let colorScale = () => {};

	//d3 projection
	const projection = geoNaturalEarth1().fitSize([width, height], sphere);
	const path = geoPath(projection);

	//OLD CODE
	//fetch both json file
	// json(
	// 	'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	// ).then((data1) => {
	// 	//data has all the information of countries: names, id, geomatrical data, etc.
	// 	//then save all that into an array named "dataset"
	// 	dataset = data1.features;
	// 	json('https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.json').then((data2) => {
	// 		for(let key in data2){
	// 			dataset2.push(data2[key]);
	// 		}
	// 		//add energy data into the geojson file, under properties
	// 		//some countries will be missing energy data due to the missing ID
	// 		dataset.forEach((i) => {
	// 			dataset2.forEach((j) => {
	// 				if (i.id == j.iso_code){
	// 					i.properties.data = j.data
	// 				}
	// 			})
	// 		})
	// 		//console.log(dataset)
	// 			const nameExtent = extent(dataset, d => d.properties.name.length);
	// 			colorScale = scaleLinear().domain(nameExtent).range(["white", "black"])

	// 	})

	// });

	//fetch the geographical data from geojson, process with d3
	json(
		'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	).then((data1) => {
		//data has all the information of countries: names, id, geomatrical data, etc.
		//save all that into an array named "dataset"
		dataset = data1.features;

		//fetch energy data from a csv file, process with d3
		csv('https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv').then((data2) => {

			//use the year in data to create a dropdown
			let dropdown = document.getElementById('yearSelect');
			data2.forEach((i) => {
				let option = document.createElement("option");
				if (i.iso_code == "AFG"){
					option.value = i.year;
					option.text = i.year;
					dropdown.append(option);
				}
			})
			
			//add population data based on the year selected from the dropdown
			dropdown.addEventListener('change', () => {
				let currentSelect = dropdown.value;
				dataset.forEach((i) => {
					data2.forEach((j) => {
						if (i.id == j.iso_code && currentSelect == j.year) {
							i.properties.data = j.population;
						}
					})
				})
				//change the color scaling based on the year selected from the dropdown
				const numExtent = extent(dataset, d => d.properties.data);
				colorScale = scaleLinear().domain(numExtent).range(["red", "white"])
			})
			
			console.log(dataset)

		})
	})


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
	<!-- draw the map  -->
	<svg viewBox="0 0 {width} {height}">
		{#each dataset as data}
			<path
				class="feature-path"
				transition:draw={{ duration: 2000, delay: 0, easing: quadInOut }}
				d = {path(data)}
				on:mousemove={handleMousemove(data)}
				fill = {colorScale(data.properties.data)}
			/>
		{/each}
	</svg>

	<!-- Using a dropdown for now -->
	<select id = "yearSelect" >	</select>
	
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
		stroke: rgb(6, 52, 70);
		stroke-width: 1.5px;
	}
</style>
