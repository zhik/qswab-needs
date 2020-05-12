<script>
    import {onMount} from 'svelte'
    import {needsData} from '../stores'

    let container;
    let map

    onMount(() => {
        mapboxgl.accessToken = 'pk.eyJ1IjoiemhpayIsImEiOiJjaW1pbGFpdHQwMGNidnBrZzU5MjF5MTJiIn0.N-EURex2qvfEiBsm-W9j7w';
        map = new mapboxgl.Map({
            container,
            style: 'mapbox://styles/zhik/cka343h1701vn1imnfp2nptkj',
            center: [-73.8217, 40.67],
            zoom: 10.3

        });
    })


    $: {
        if (map && $needsData && $needsData.features) {
            map.on('load', () => {
                map.addSource('needs', {'type': 'geojson', data: $needsData});

                map.addLayer({
                    'id': 'needs-streets',
                    'type': 'line',
                    'source': 'needs',
                    'paint': {
                        'line-color': '#363636',
                        'line-opacity': 0.8,
                        'line-width': 3
                    },
                    'filter': ['==', '$type', 'LineString']
                });

                map.on('click', 'needs-streets', function (e) {
                    const feature = e.features[0]
                    const {type, request, explanation, tracking_code} = feature.properties
                    const district = tracking_code.slice(0, 3)

                    new mapboxgl.Popup()
                            .setLngLat(e.lngLat)
                            .setHTML(`<h3>${request}</h3><p>${district}</p><p>${explanation}</p>`)
                            .addTo(map);
                });

                map.on('mouseenter', 'needs-streets', function () {
                    map.getCanvas().style.cursor = 'pointer';
                });

                map.on('mouseleave', 'needs-streets', function () {
                    map.getCanvas().style.cursor = '';
                });

                //for each point feature add html marker
                $needsData.features
                        .filter(feature => feature.geometry.type === 'Point')
                        .forEach(feature => {
                            // create a HTML element for each feature
                            const el = document.createElement('div');
                            const {type, request, explanation, tracking_code} = feature.properties
                            const district = tracking_code.slice(0, 3)
                            el.className = `marker-${type}`;
                            el.textContent = feature.properties.revised_priority

                            // make a marker for each feature and add to the map
                            new mapboxgl.Marker(el)
                                    .setLngLat(feature.geometry.coordinates)
                                    .setPopup(new mapboxgl.Popup({offset: 5})
                                            .setHTML(`<h3>${request}</h3><p>${district}</p><p>${explanation}</p>`))
                                    .addTo(map);
                        })
            });
        }
    }

</script>

<div class="map-view">
    <div id="map" bind:this={container}></div>
    <div class="legend">
        <h4>Queens DSNY Needs</h4>
        <div class="legend-item">
            <div class="marker-district">1</div>
            <div class="label">District-wide need</div>
        </div>
        <div class="legend-item">
            <div class="marker-location">1</div>
            <div class="label">Location-specific need</div>
        </div>
    </div>
</div>

<style>
    .map-view{
        position: relative;
    }

    #map {
        width: 100%;
        height: 700px;
    }

    .legend{
        line-height: 18px;
        color: #555;
        position: absolute;
        top: 10px;
        right: 20px;
        z-index: 2;
        background-color: rgba(255, 255, 255, 0.7);
        padding: 5px 10px;
    }

    .legend h4{
        margin: 5px 0px;
    }

    .legend-item {
        margin-right: 1rem;
        display: flex;
        flex-direction: row;
    }

    .label {
        margin-left: 1rem;
    }

    :global(.marker-district) {
        text-align: center;
        background-color: #363636;
        color: white;
        font-weight: 800;
        line-height: 16.2px;
        height: 16px;
        width: 16px;
        font-size: 10px;
        border-radius: 50%;
        cursor: pointer;
    }

    :global(.marker-location) {
        text-align: center;
        background-color: #363636;
        color: white;
        font-weight: 800;
        line-height: 16.2px;
        height: 16px;
        width: 16px;
        font-size: 10px;
        cursor: pointer;
    }

    :global(.mapboxgl-popup) {
        max-width: 300px;
    }

    :global(.mapboxgl-popup-content) {
        font-family: 'Open Sans', sans-serif;
        max-height: 300px;
        overflow-y: auto;
        padding: 5px 10px !important;
    }

</style>