<script>
	import { setContext, createEventDispatcher, onDestroy } from "svelte";
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
	export let locate = false;
	export let tabbable = false;

	export let zoom = null;
	export let center = null;
	export let interactive = true;

	let container;
	let _options = {};
	let loaded = false;

	setContext("map", {
		getMap: () => map,
	});
	
	// Interpret location
	if (location.bounds) {
		_options.bounds = location.bounds;
	} else if (typeof location.lng == 'number' && typeof location.lat == 'number') {
		_options.center = [location.lng, location.lat];
		if (typeof location.zoom == 'number') {
			_options.zoom = location.zoom;
		}
	}
	_options = {..._options, ...options}; // Combine core options + custom user options

	function load() {
		map = new maplibre.Map({
			container,
			style,
			minZoom: minzoom,
			maxZoom: maxzoom,
			interactive,
			..._options,
		});
		
		if (controls) {
			map.addControl(new maplibre.NavigationControl({showCompass: false}));
		}
		
		if (locate) {
			map.addControl(new maplibre.GeolocateControl());
		}
		
		// Get initial zoom level
		map.on("load", (e) => {
			zoom = map.getZoom();
			center = map.getCenter();
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
		});
	};

	// Function to switch map style if style prop changes
	function setStyle(style) {
		if (map) map.setStyle(style);
		dispatch("style", {
			style
		});
	}
	$: setStyle(style);

	onDestroy(() => {
		if (map) map.remove();
		map = null;
	});
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://unpkg.com/maplibre-gl@2.1.9/dist/maplibre-gl.css"
		on:load={load}
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