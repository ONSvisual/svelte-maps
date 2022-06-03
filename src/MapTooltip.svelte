<script>
  import { getContext, onDestroy } from 'svelte';
  import maplibre from 'maplibre-gl';

  export let content;

  const tooltip = new maplibre.Popup({
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

  onDestroy(() => tooltip.remove());
</script>

<style>
  :global(.maplibregl-popup-content) {
		padding: 5px 10px !important;
	}
</style>