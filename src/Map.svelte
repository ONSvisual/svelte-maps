<script>
	import { setContext, createEventDispatcher, onMount, onDestroy } from "svelte";
	import maplibre from "maplibre-gl";

	const dispatch = createEventDispatcher();

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
		"layers": [
			{
				"id": "background",
				"type": "background",
				"paint": {"background-color": "lightgrey"}
			}
		]
	}; // Can be a json style definition or a url
	export let options = {};
	export let minzoom = 0;
	export let maxzoom = 14;
	export let controls = false;
	export let tabbable = false;

	export let zoom = null;
	export let center = null;
	export let pitch = null;
	export let bearing = null;
	export let interactive = true;
	export let attribution = true;

	let container;
	let _options = {};
	let loaded = false;

	setContext("map", {
		getMap: () => map,
	});

	function sleep (ms = 1000) {
  	return new Promise((resolve) => setTimeout(resolve, ms));
	}
	
	// Interpret location
	if (location.bounds) {
		_options.bounds = location.bounds;
	} else if (location.lng && location.lat) {
		_options.center = [+location.lng, +location.lat];
		if (location.zoom) {
			_options.zoom = +location.zoom;
		}
		if (location.pitch) {
			_options.pitch = +location.pitch;
		}
		if (location.bearing) {
			_options.bearing = +location.bearing;
		}
	}
	// Disable attribution if attribution = false
	if (!attribution) {
		_options.attributionControl = false;
	}

	_options = {..._options, ...options}; // Combine core options + custom user options

	onMount(() => {
		map = new maplibre.Map({
			container,
			style,
			minZoom: minzoom,
			maxZoom: maxzoom,
			interactive,
			..._options,
		});
		
		if (controls && !Array.isArray(controls)) {
			map.addControl(new maplibre.NavigationControl({showCompass: false}));
		} else if (Array.isArray(controls) && controls != ["locate"]) {
			map.addControl(new maplibre.NavigationControl({showCompass: controls.includes("compass"), visualizePitch: controls.includes("pitch")}));
		}
		
		if (Array.isArray(controls) && controls.includes("locate")) {
			map.addControl(new maplibre.GeolocateControl());
		}
		
		// Get initial zoom level
		map.on("load", (e) => {
			zoom = map.getZoom();
			center = map.getCenter();
			pitch = map.getPitch();
			bearing = map.getBearing();
			loaded = true;
			
			// Prevent map from being tabbable
			if (!tabbable && document.querySelector(`#${id} canvas`)) {
				document.querySelector(`#${id} canvas`).tabIndex = "-1";
			}

			dispatch("load", {
				event: e
			});
		});

		// Update zoom level and center when the view changes
		map.on("moveend", () => {
			zoom = map.getZoom();
			center = map.getCenter();
			pitch = map.getPitch();
			bearing = map.getBearing();
		});
	});

	onDestroy(async () => {
		await sleep(250);

		if (map) map.remove();
		map = null;
	});

	// Function to switch map style if style prop changes
	function setStyle(style) {
		if (map) map.setStyle(style);
		dispatch("style", {
			style
		});
	}
	$: setStyle(style);
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://unpkg.com/maplibre-gl@2.1.9/dist/maplibre-gl.css"
	/>
</svelte:head>

<div bind:this={container} {id} class="map">
	{#if loaded}
		<slot />
	{/if}
</div>

<style>
	:global(.maplibregl-control-container button) {
		margin: 0;
	}
	.map {
		width: 100%;
		height: 100%;
	}
</style>