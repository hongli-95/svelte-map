<script>
	import { json, csv, max, min} from 'd3';
	import { geoPath, geoNaturalEarth1, scaleLinear, extent } from 'd3';
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

	//fetch the geographical data from geojson, process with d3
	json(
		'https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson'
	).then((data1) => {
		//data has all the information of countries: names, id, geomatrical data, etc.
		//save all that into an array named "dataset"
		dataset = data1.features;

		//fetch energy data from a csv file, process with d3
		csv('https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv').then((data2) => {

			
			let slider = document.getElementById('yearSelect');
			let years = [];

			//get the range of years from any country
			data2.forEach((i) => {
				if (i.iso_code == "AFG"){
					years.push(i.year);
				}
			})

			//set the min and max for the slider
			slider.max = Math.max(...years);
			slider.min = Math.min(...years);
			slider.value = Math.min(...years);
			
			//add population data based on the year selected from the slider
			slider.addEventListener('input', () => {
				let currentSelect = slider.value;
				dataset.forEach((i) => {
					data2.forEach((j) => {
						if (i.id == j.iso_code && j.year == currentSelect) {
							//j.propertyName determines what data is pulled from the csv file
							//Using population data for now for the sake of simplicity 
							//Will probably add all the data from the csv at the end, and use a slider to choose which data to display
							i.properties.data = j.population;
						}
					})
				})
				//change the color scaling based on the year selected from the slider
				//d3.extent compares using natural order instead of numeric order, so parseInt is implemented
				//WORKED!!!
				const numExtent = extent(dataset, d => parseInt(d.properties.data));
				colorScale = scaleLinear().domain(numExtent).range(["white", "red"]);
			})
		})
	})

	function handleMousemove() {
		return function handleMousemoveFn() {
			//raise: Layercake component(?) Used to handle borders of neighboring countries
			raise(this);
		}
	}

</script>


<main>
	<!-- draw the map  -->
	<div id="mapCanvas">
		<svg viewBox = "0 0 {width} {height}">
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
	</div>

	<br>

	<!-- Using a slider to select which year -->
	<div id="select">
		<input type="range" id="yearSelect">
	</div>
	
	
</main>


<style>
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	/* styling for the svg container */
	#mapCanvas {
		width: 90%;
		height: 80%;
		display: block;
		margin: auto;
	}

	/* styling for the silder */
	#yearSelect {
		width: 80%;
		display: block;
		margin: auto;
		-webkit-appearance: none;  /* Override default CSS styles */
		height: 15px;
		background: #d3d3d3;
		opacity: 0.5;
		transition: opacity .2s;
		border-radius: 10px;
	}

	#yearSelect:hover {
		opacity: 1;
	}

	path {
		stroke: gray;
	}

	.feature-path:hover {
		stroke: rgb(6, 52, 70);
		stroke-width: 1.5px;
	}
</style>
