<script>
	//This file handles the map drawing part. 
	//called from the app.svelte, passed in the dataset array, and draw each country
	//if other data is wanted other than just the map, such as countries names, it might be handled here as well.

	import { geoPath, geoNaturalEarth1 } from 'd3';
	import { createEventDispatcher } from 'svelte';
	import { raise } from 'layercake';
	import { draw } from 'svelte/transition';
	import { quadInOut } from 'Svelte/easing';

	export let dataset = [];

	const sphere = { type: 'Sphere' };
	var height = window.innerHeight * 0.95;
	var width = window.innerWidth * 0.95;

	const projection = geoNaturalEarth1().fitSize([width, height], sphere);

	const path = geoPath(projection);

	//svelte event dispatcher
	const dispatch = createEventDispatcher();

	function handleMousemove(feature) {
		return function handleMousemoveFn(e) {
			//raise: Layercake component(?) Used to handle borders of neighboring countries
			raise(this);
			if (e.layerX !== 0 && e.layerY !== 0) {

				//could be named however 
				//props function unknown, seems to be used to access what is being passed in here from other svelte file
				dispatch('eventname', {e, props: feature.properties.name })
			}
		}
	}
</script>

<!-- draw each country based on the set dataset from App.svelte -->
{#each dataset as data}
	<path
		class="feature-path"
		transition:draw={{ duration: 2000, delay: 0, easing: quadInOut }}
		d={path(data)}
		on:mousemove={handleMousemove(data)}
	/>
{/each}

<style>
	path {
		fill: greenyellow;
		stroke: gray;
	}

	.feature-path:hover {
		stroke: rgb(255, 255, 255);
		stroke-width: 1.5px;
	}
</style>
