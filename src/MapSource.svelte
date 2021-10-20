<script>
	import { getContext, setContext } from 'svelte';
	
	export let id;
	export let type;
	export let url = null;
	export let props = {};
	export let data = null;
	export let layer = null;
	export let promoteId = null;
	export let minzoom = null;
	export let maxzoom = null;
	
	let loaded = false;
	
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

	function isSourceLoaded(){
    if (map.isSourceLoaded(id)) {
			loaded = true;
			console.log(id + ' map source loaded!');
    } else {
			setTimeout(() => {
				console.log('...');
				isSourceLoaded();
			}, 250);
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
		console.log(id + ' map source loading...');
		let layerdef;
		
  	if (type == "geojson") {
	  	if (data) {
		  	layerdef = {
	  		  type: type,
	  		  data: data,
					...props
				};
  		} else if (url) {
	  		layerdef = {
	  		  type: type,
	  		  data: url,
					...props
				};
		  }
	  } else if (type == "vector") {
	  	layerdef = {
	  		type: type,
	  		tiles: [ url ],
	  		...props
			};
		} else if (type == "raster") {
	  	layerdef = {
	  		type: type,
	  		tiles: [ url ],
				tileSize: tilesize,
	  		...props
			};
		}
		if (layerdef) {
			map.addSource(id, layerdef);
			isSourceLoaded();
		}
	};

	addSource();

	function setData() {
		map.getSource(id).setData(data);
	}

	$: loaded && data && setData();
	
</script>

{#if loaded}
	<slot></slot>
{/if}