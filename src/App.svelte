<script>
	import { json, csv} from 'd3';
	import { geoPath, geoNaturalEarth1, scaleLinear, extent } from 'd3';
	import { raise } from 'layercake';
	import { draw } from 'svelte/transition';
	import { quadInOut } from 'Svelte/easing';

	let drawThis = [];

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
	).then((geoData) => {
		//data has all the information of countries: names, id, geomatrical data, etc.
		//save all that into an array named "drawThis"
		drawThis = geoData.features;

		//fetch energy data from a csv file, process with d3
		//https://critviz.s3.amazonaws.com/uploads/user_file/file/191492/owid-energy-data_cleaned2.csv
		//https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv
		csv('https://critviz.s3.amazonaws.com/uploads/user_file/file/191492/owid-energy-data_cleaned2.csv').then((energyData) => {

			let yearLabel = document.getElementById('yearLabel');
			let slider = document.getElementById('yearSelect');
			let dropDown = document.getElementById('catSelect');
			let years = [];
			let dropDownSelect;

			energyData.forEach((i) => {
				if (i.iso_code == "AFG"){
					//get the range of years from any single country
					years.push(i.year);

					//add categories to compare, based on the categories from any single country, in a random year(1900 ~ 2020)
					if (i.year == "1999") {
						for (let propertyName in i) {
							if (propertyName == "per_capita_electricity") {
								let x = document.createElement("option")
								x.text = "Electricity Per Capita"
								dropDown.appendChild(x)
							} else if (propertyName == "renewables_elec_per_capita") {
								let x = document.createElement("option")
								x.text = "Renewables Electricity Per Capita"
								dropDown.appendChild(x)
							} else if (propertyName == "fossil_cons_per_capita") {
								let x = document.createElement("option")
								x.text = "Fossil Consumption Per Capita"
								dropDown.appendChild(x)
							}
						}
					}
				} 
			})

			//set the min and max for the slider
			slider.max = Math.max(...years);
			slider.min = 1900;
			slider.value = slider.min;
			yearLabel.textContent = slider.value;

			//event listener for the dropdown 
			dropDown.addEventListener('change', () => {
				let currentSelect = dropDown.options[dropDown.selectedIndex].text
				switch (currentSelect) {
					case "Fossil Consumption Per Capita":
						dropDownSelect = "fossil_cons_per_capita"
						break;
					case "Renewables Electricity Per Capita":
						dropDownSelect = "renewables_elec_per_capita"
						break;
					case "Electricity Per Capita":
						dropDownSelect = "per_capita_electricity"
						break;
				}
			})
			
			//add selected data based on the year selected from the slider
			slider.addEventListener('input', () => {
				let currentSelect = slider.value;
				drawThis.forEach((i) => {
					let yearArray = []
					energyData.forEach((j) => {
						if (i.id == j.iso_code) {
							yearArray.push(j.year);
							if (yearArray.includes(currentSelect)) {
								i.properties.data = parseInt(j[`${dropDownSelect}`]);
							}
							else {
								delete i.properties.data
							}
						}
					})
				})
				//change the color scaling based on the year selected from the slider
				//d3.extent compares using natural order instead of numeric order, so parseInt (above) is implemented
				const numExtent = extent(drawThis, d => d.properties.data);
				colorScale = scaleLinear().domain(numExtent).range(["white", "red"]);
				yearLabel.textContent = slider.value
				//console.log(numExtent)
				//console.log(drawThis)
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
		<label id="yearLabel" for=""></label>

		<select id="catSelect">
			<option value="" disabled selected hidden>Select a Category below...</option>
		</select>

		<svg viewBox = "0 0 {width} {height}">
			{#each drawThis as eachCountry}
				<path
					class="feature-path"
					transition:draw={{ duration: 1000, delay: 0, easing: quadInOut }}
					d = {path(eachCountry)}
					on:mousemove={handleMousemove(eachCountry)}
					fill = {colorScale(eachCountry.properties.data)}
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
		width: 100%;
		height: 80%;
		display: block;
		margin: auto;
	}

	svg {
		width: 90%;
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

	#yearLabel {
		width: 15%;
		text-align: left;
		position: absolute;
		font-size: xx-large;
		font-family: Verdana, sans-serif;
	}

	#catSelect {
		width: 18%;
		position: absolute;
		text-align: left;
		margin-top: 3em;
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
