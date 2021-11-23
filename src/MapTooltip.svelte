<script>
  import { getContext } from 'svelte';
  import mapbox from './lib/mapbox-gl@1.13.2';
	// Mapbox source code is bundled due to versioning & ES6 import issues
	// https://cdn.skypack.dev/-/mapbox-gl@v1.13.2-asizChmwkQobquJNQjgb/dist=es2020,mode=imports,min/optimized/mapbox-gl.js

  export let content;

  const tooltip = new mapbox.Popup({
		closeButton: false,
		closeOnClick: false
	});

  const { getMap } = getContext('map');
	const map = getMap();
  const hoverObj = getContext('hover');

  function updateTooltip(obj, content) {
    if (obj.id) {
      tooltip
			.setLngLat(obj.event.lngLat)
      .setHTML(content ? content : obj.code)
      .addTo(map);
    } else {
      tooltip.remove();
    }
  }

  $: updateTooltip($hoverObj, content);
</script>

<style>
  :global(.mapboxgl-popup-content) {
		padding: 5px 10px !important;
	}
</style>