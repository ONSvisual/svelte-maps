<script>
	import { getContext, setContext, createEventDispatcher } from 'svelte';
	import { writable } from 'svelte/store';

	const dispatch = createEventDispatcher();
	
	export let id;
	export let type;
	export let filter = null;
	export let layout = {};
	export let paint = {};
	export let data = null;
	export let colorKey = "color";
	export let nameKey = null;
	export let valueKey = null;
	export let idKey = null;
	export let select = false;
	export let clickIgnore = false;
	export let clickCenter = false;
	export let selected = null;
	export let hover = false;
	export let hovered = null;
	export let highlight = false;
	export let highlightKey = 'highlighted';
	export let highlighted = [];
	export let order = null;
	export let maxzoom = null;
	export let minzoom = null;
	export let sourceLayer = null;
	
	const { source, layer, promoteId } = getContext('source');
	const { getMap } = getContext('map');
	const map = getMap();

	setContext("layer", {
		layer: id
	});

	const hoverObj = writable({
		id: null,
		feature: null,
		event: null
	});
	setContext("hover", hoverObj);

	idKey = idKey ? idKey : promoteId;
	sourceLayer = sourceLayer ? sourceLayer : layer;
	
	let selectedPrev = null;
	let hoveredPrev = null;
	let highlightedPrev = [];
	
	if (map.getLayer(id)) {
    map.removeLayer(id);
	}
	
	let options = {
		'id': id,
		'type': type,
		'source': source,
		'paint': paint,
		'layout': layout
	};

	if (filter) {
		options['filter'] = filter;
	}
	
	if (sourceLayer) {
		options['source-layer'] = sourceLayer;
	}
	if (maxzoom) {
		options['maxzoom'] = maxzoom;
	}
	if (minzoom) {
		options['minzoom'] = minzoom;
	}
	
	map.addLayer(options, order);

	// Updates "color" feature states for all geo codes
	// Assumes that each data point has the colours defined on the colorCode key
	function updateColors() {
		console.log('updating colors...');

		data.forEach(d => {
			map.setFeatureState({
				source: source,
				sourceLayer: sourceLayer,
				id: d[idKey]
			}, {
				color: colorKey ? d[colorKey] : null,
				name: nameKey ? d[nameKey] : null,
				value: valueKey ? d[valueKey] : null
			});
		});
	}

	$: data && (data || colorKey) && updateColors();
	
	// Updates the "highlighted" feature state as geo codes are added to/removed from the highlighted array
	$: if (highlight && highlighted != highlightedPrev) {
		if (highlightedPrev[0]) {
			highlightedPrev.forEach(id => {
				let state = {};
				state[highlightKey] = false;
				map.setFeatureState(
					{ source, sourceLayer, id },
					state
				);
			});
		}
		highlightedPrev = highlighted;
		if (highlighted[0]) {
			highlighted.forEach(id => {
				let state = {};
				state[highlightKey] = true;
				map.setFeatureState(
					{ source, sourceLayer, id },
					state
				);
			});
		}
	}
	
	// Adds a click event to change the selected geo code (if select = true for map layer)
	if (select) {
		map.on('click', id, (e) => {
      if (e.features.length > 0 && !clickIgnore) {
				let feature = e.features[0];
				selected = feature.id;

				dispatch('select', {
					id: selected,
					feature: feature,
					event: e
				});
				
				if (selectedPrev) {
					map.setFeatureState(
            { source: source, sourceLayer: sourceLayer, id: selectedPrev },
            { selected: false }
          );
				}
				
				map.setFeatureState(
          { source: source, sourceLayer: sourceLayer, id: selected },
          { selected: true }
				);

				if (clickCenter) {
					let center = centroid(e.features[0].toJSON().geometry);
					map.flyTo({
						center: center.geometry.coordinates
					});
				}
				
				selectedPrev = selected;
			}
    });
	}
	
	// Updates the selected geo code if it is changed elsewhere in the app (outside of this component)
	$: if (select && selected != selectedPrev) {
		if (selectedPrev) {
			map.setFeatureState(
				{ source: source, sourceLayer: sourceLayer, id: selectedPrev },
				{ selected: false }
			);
		}
		if (selected) {
			map.setFeatureState(
				{ source: source, sourceLayer: sourceLayer, id: selected },
				{ selected: true }
			);
		}
		selectedPrev = selected;
	}
	
	// Adds an event to update the hovered geo code when the mouse is moved over the map
	if (hover) {
		map.on('mousemove', id, (e) => {
      if (e.features.length > 0) {
				if (hovered) {
          map.setFeatureState(
            { source: source, sourceLayer: sourceLayer, id: hovered },
            { hovered: false }
          );
        }
				let feature = e.features[0];
				hovered = hoveredPrev = feature.id;

				hoverObj.set({
					id: hovered,
					feature: feature,
					event: e
				});

				dispatch('hover', $hoverObj);

        map.setFeatureState(
          { source: source, sourceLayer: sourceLayer, id: hovered },
          { hovered: true }
        );

        // Change the cursor style as a UI indicator.
				map.getCanvas().style.cursor = 'pointer';
      }
		});
		
		map.on('mouseleave', id, (e) => {
			if (hovered) {
        map.setFeatureState(
          { source: source, sourceLayer: sourceLayer, id: hovered },
          { hovered: false }
				);
      }
			hovered = hoveredPrev = null;

			hoverObj.set({
				id: null,
				feature: null,
				event: e
			});

			dispatch('hover', $hoverObj);
			
			// Reset cursor and remove popup
			map.getCanvas().style.cursor = '';
    });
	}
	
	// Updates the hovered geo code if it is changed elsewhere in the app (outside of this component)
	$: if (hover && hovered != hoveredPrev) {
		if (hoveredPrev) {
			map.setFeatureState(
				{ source: source, sourceLayer: sourceLayer, id: hoveredPrev },
        { hovered: false }
			);
		}
		if (hovered) {
			map.setFeatureState(
				{ source: source, sourceLayer: sourceLayer, id: hovered },
        { hovered: true }
			);
		}
		hoveredPrev = hovered;
	}
	
</script>

{#if hover}
<slot {hovered}></slot>
{/if}