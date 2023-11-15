<script>
	import { getContext, setContext, onMount, onDestroy } from 'svelte';
	
	export let id;
	export let type;
	export let url = null;
	export let props = {};
	export let data = null;
	export let layer = null;
	export let promoteId = null;
	export let minzoom = null;
	export let maxzoom = null;
	export let tilesize = 256;
	
	let loaded = false;
	let urlPrev = url;
	
	const { getMap } = getContext('map');
	const map = getMap();

	setContext("source", {
		source: id,
		layer: layer,
		promoteId: promoteId
	});
	
	if (map.getSource(id)) {
    map.removeSource(id);
	}

	function sleep (ms = 1000) {
		return new Promise((resolve) => setTimeout(resolve, ms));
	}

	async function isSourceLoaded(){
		await sleep(100);

    if (map.isSourceLoaded(id)) {
			loaded = true;
			console.debug(id + ' map source loaded!');
    } else {
			console.debug('...');
			isSourceLoaded();
		}
	}
	
	// Set optional source properties
	if (minzoom) {
    props.minzoom = minzoom;
	}
	if (maxzoom) {
    props.maxzoom = maxzoom;
	}
	if (layer && promoteId) {
		props.promoteId = {};
		props.promoteId[layer] = promoteId;
	} else if (promoteId) {
		props.promoteId = promoteId;
	}
	
	function addSource() {
		console.debug(id + ' map source loading...');
		let layerdef;
		
  	if (type == "geojson") {
	  	if (data) {
		  	layerdef = {
	  		  type,
	  		  data,
					...props
				};
  		} else if (url) {
	  		layerdef = {
	  		  type,
	  		  data: url,
					...props
				};
		  }
	  } else if (type == "vector") {
	  	layerdef = {
	  		type,
				tiles: [url],
	  		...props
			};
		} else if (type == "raster") {
	  	layerdef = {
	  		type,
	  		tiles: [ url ],
				tileSize: tilesize,
	  		...props
			};
		} else if (type == "raster-dem") {
			layerdef = {
				type,
				tiles: [ url ],
				tileSize: tilesize,
				...props
			}
		}
		if (layerdef) {
			map.addSource(id, layerdef);
			isSourceLoaded();
		}
	};

	function setData(data) {
		let source = map.getSource(id);
		if (source) source.setData(data);
	}
	$: type == "geojson" && loaded && setData(data);

	function setVectorTiles(url) {
		if (url !== urlPrev) {
			let source = map.getSource(id);
			if (source) source.setTiles([url]);
			urlPrev = url;
		}
	}
	$: type == "vector" && loaded && setVectorTiles(url);

	function setRasterTiles(url) {
		if (url !== urlPrev) {
			map.getSource(id).tiles = [ url ];
			map.style.sourceCaches[id].clearTiles();
			map.style.sourceCaches[id].update(map.transform);
			map.triggerRepaint();
			urlPrev = url;
		}
	}
	$: type == "raster" && loaded && setRasterTiles(url);
	
	onMount(addSource);

	onDestroy(async () => {
		if (typeof map?.getSource === "function" && map.getSource(id)) {
			let layers = map.getStyle().layers;
			layers.filter(l => l.source == id)
			.forEach(l => {
				map.removeLayer(l.id);
			});

			map.removeSource(id);
		}
	});
</script>

{#if loaded}
	<slot></slot>
{/if}