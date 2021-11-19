<script>
	import { onMount, setContext } from "svelte";
	import mapbox from "./lib/mapbox-gl@1.13.2";
	// Mapbox source code is bundled due to versioning & ES6 import issues
	// https://cdn.skypack.dev/-/mapbox-gl@v1.13.2-asizChmwkQobquJNQjgb/dist=es2020,mode=imports,min/optimized/mapbox-gl.js

	export let map;
	export let id = "map";
	export let location = {
		lng: 15,
		lat: 45,
		zoom: 1
	};
	export let style = {
		"version": 8,
		"sources": {},
		"layers": []
	};
	export let minzoom = 0;
	export let maxzoom = 14;
	export let controls = false;
	export let locate = false;
	export let tabbable = false;

	export let zoom = null;
	export let center = null;
	export let interactive = true;

	let container;
	let w;
	let h;
	let options;
	let loaded = false;

	setContext("map", {
		getMap: () => map,
	});
	
	// Interpret location
	if (location.bounds) {
		options = { bounds: location.bounds };
	} else if (typeof location.lng == 'number' && typeof location.lat == 'number') {
		options = {
			center: [location.lng, location.lat],
		};
		if (typeof location.zoom == 'number') {
			options.zoom = location.zoom;
		}
	}

	onMount(() => {
		const link = document.createElement("link");
		link.rel = "stylesheet";
		link.href = "https://unpkg.com/mapbox-gl@1.13.2/dist/mapbox-gl.css";

		link.onload = () => {
			map = new mapbox.Map({
				container,
				style,
				minZoom: minzoom,
				maxZoom: maxzoom,
				interactive,
				...options,
			});
			
			if (controls) {
				map.addControl(new mapbox.NavigationControl({showCompass: false}));
			}
			
			if (locate) {
				map.addControl(new mapbox.GeolocateControl());
			}
			
			// Get initial zoom level
			map.on("load", () => {
				zoom = map.getZoom();
				center = map.getCenter();
				loaded = true;
				
				// Prevent map from being tabbable
				if (!tabbable && document.querySelector(`#${id} canvas`)) {
					document.querySelector(`#${id} canvas`).tabIndex = "-1";
				}
			});

			// Update zoom level and center when the view changes
			map.on("moveend", () => {
				zoom = map.getZoom();
				center = map.getCenter();
			});
		};

		document.head.appendChild(link);

		return () => {
			map.remove();
			link.parentNode.removeChild(link);
		};
	});

	// Function that forces map to resize to fit parent div, in case it doesn't automatically
	function resizeCanvas() {
		if (loaded) {
			let canvas = document.getElementsByClassName("mapboxgl-canvas");
		  if (canvas[0]) {
			  canvas[0].style.width = "100%";
			  canvas[0].style.height = "100%";
			  map.resize();
		  }
		}
	}

	// Invoke above function when parent div size changes
	$: (w || h) && resizeCanvas();
</script>

<div bind:clientWidth={w} bind:clientHeight={h} bind:this={container} {id}>
	{#if loaded}
		<slot />
	{/if}
</div>

<style>
	:global(.mapboxgl-control-container button) {
		margin: 0;
	}
	div {
		width: 100%;
		height: 100%;
	}
</style>