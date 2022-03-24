<script>
	import { geoPath, geoNaturalEarth1 } from 'd3';
	import { draw } from 'svelte/transition';
	import { quadInOut } from 'Svelte/easing';

	export let dataset = [];

	const sphere = { type: 'Sphere' };
	var height = window.innerHeight * 0.95;
	var width = window.innerWidth * 0.95;

	const projection = geoNaturalEarth1().fitSize([width, height], sphere);

	const path = geoPath(projection);
</script>

{#each dataset as data}
	<path
		transition:draw={{ duration: 2000, delay: 0, easing: quadInOut }}
		d={path(data)}
	/>
{/each}

<style>
	path {
		fill: greenyellow;
		stroke: gray;
		width: 100%;
		height: auto;
	}
</style>
