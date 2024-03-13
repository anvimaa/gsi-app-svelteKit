<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	let map: L.Map | undefined;
	let mapElement: HTMLElement;

	onMount(async () => {
		if (browser) {
			map = L.map(mapElement).setView([51.505, -0.09], 13);

			//-- Definir os mapas
			let osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
				maxZoom: 19,
				crossOrigin: true
			});

			let googleSat = L.tileLayer('http://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
				maxZoom: 20,
				subdomains: ['mt0', 'mt1', 'mt2', 'mt3'],
				crossOrigin: true
			});

			let googleTraffic = L.tileLayer(
				'https://{s}.google.com/vt/lyrs=m@221097413,traffic&x={x}&y={y}&z={z}',
				{
					maxZoom: 20,
					minZoom: 2,
					subdomains: ['mt0', 'mt1', 'mt2', 'mt3'],
					crossOrigin: true
				}
			);
			//-- Difinir os Mapas

			// Definir as camadas
			let baseLayers = {
				'Ver como Satálite': googleSat,
				'Ver como Open Street': osm,
				'Ver como Tráfego': googleTraffic
			};

			// Apresentar o mapa
			googleSat.addTo(map);

			// Export Button
			var showHome = `<a href="/" title="Voltar" type="button" class="btn btn-sm variant-filled">Dashboard</a>`;
			var showHomeButton = new L.Control({ position: 'bottomleft' });
			showHomeButton.onAdd = function (map) {
				this._div = L.DomUtil.create('div');
				this._div.innerHTML = showHome;
				return this._div;
			};
			showHomeButton.addTo(map);

			// Adicionar controle de camadas ao mapa
			L.control.layers(baseLayers).addTo(map);

			var editableLayers = new L.FeatureGroup();
			map.addLayer(editableLayers);

			var MyCustomMarker = L.Icon.extend({
				options: {
					shadowUrl: null,
					iconAnchor: new L.Point(12, 12),
					iconSize: new L.Point(24, 24)
				}
			});

			var options = {
				position: 'topleft',
				draw: {
					polyline: {
						shapeOptions: {
							color: '#f357a1',
							weight: 2
						}
					},
					polygon: {
						allowIntersection: false, // Restricts shapes to simple polygons
						drawError: {
							color: '#e1e100', // Color the shape will turn when intersects
							message: '<strong>Atenção!<strong> não podes sobrepor a marcação!' // Message that will show when intersect
						},
						shapeOptions: {
							color: '#bada55'
						}
					},
					circle: true, // Turns off this drawing tool
					rectangle: {
						shapeOptions: {
							clickable: false
						}
					},
					marker: {
						icon: new MyCustomMarker()
					}
				},
				edit: {
					featureGroup: editableLayers, //REQUIRED!!
					remove: true
				}
			};

			var drawControl = new L.Control.Draw(options);
			map.addControl(drawControl);

			map.on('draw:created', function (e) {
				var type = e.layerType,
					layer = e.layer;

				if (type === 'marker') {
					layer.bindPopup('A popup!');
				}
				console.log();
				editableLayers.addLayer(layer);
			});
		}
	});

	onDestroy(async () => {
		if (map) {
			console.log('Unloading Leaflet map.');
			map.remove();
		}
	});
</script>

<main>
	<div bind:this={mapElement}></div>
</main>

<style lang="postcss">
	main div {
		height: 90vh;
	}
</style>
