<script>
	import { geoPath, geoNaturalEarth1, geoAzimuthalEqualAreaRaw } from 'd3';
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

	const dispatch = createEventDispatcher();

	function handleMousemove(feature) {
		return function handleMousemoveFn(e) {
			raise(this);
			if (e.layerX !== 0 && e.layerY !== 0) {
				dispatch('mousemove', {e, props: feature.properties })
			}
		}
	}
</script>

{#each dataset as data}
	<path
		class="feature-path"
		transition:draw={{ duration: 3000, delay: 0, easing: quadInOut }}
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
