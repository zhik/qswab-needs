<script>
    import {onMount} from 'svelte'
    import {needsData} from '../stores'

    let container;
    let map

    const categories = [
        {name: 'Litter Basket Collection & Cleaning Service', color: '#aa5e58', letter: 'L'},
        {name: 'Infrastructure & Equipment', color: '#4060a3', letter: 'I'},
        {name: 'Residential Organics, Recycle, or Refuse', color: '#719a63', letter: 'R', includes: 'Residential'},
        {name: 'Illegal Posting', color: '#846397', letter: 'P'},
        {name: 'Other', color: '#959595', letter: 'O'}
    ]

    onMount(() => {
        mapboxgl.accessToken = 'pk.eyJ1IjoiemhpayIsImEiOiJjaW1pbGFpdHQwMGNidnBrZzU5MjF5MTJiIn0.N-EURex2qvfEiBsm-W9j7w';
        map = new mapboxgl.Map({
            container,
            style: 'mapbox://styles/zhik/cka343h1701vn1imnfp2nptkj',
            center: [-73.7717, 40.67],
            zoom: 10.45

        });
    })

    $: {
        if ($needsData && $needsData.features) {
            const cats = $needsData.features.reduce((a, f) => {
                const category = f.properties.category
                a[category] = a[category] === undefined ? 1 : a[category] += 1
                return a
            }, {})
            console.log(cats)
        }
    }


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
                            const {type, request, explanation, tracking_code, revised_priority, category} = feature.properties
                            const district = tracking_code.slice(0, 3)
                            el.className = `marker marker-${type}`;
                            el.textContent = revised_priority
                            el.style.backgroundColor = (() => {
                                const found = categories.find((cat) => {
                                    if('includes' in cat){
                                        return category.includes(cat.includes)
                                    }else{
                                        return cat.name === category
                                    }
                                })
                                return found ? found.color : '#6a6a6a'
                            })()

                            // make a marker for each feature and add to the map
                            new mapboxgl.Marker(el)
                                    .setLngLat(feature.geometry.coordinates)
                                    .setPopup(new mapboxgl.Popup({offset: 5})
                                            .setHTML(`
                                                    <h3>${request}</h3>
                                                    <p><strong>${category}</strong></p>
                                                    <p><strong>Community District:</strong> ${district}</p>
                                                    <p>${explanation}</p>
                                            `))
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
            <div class="marker marker-district">1</div>
            <div class="label">District-wide need</div>
        </div>
        <div class="legend-item">
            <div class="marker marker-location">1</div>
            <div class="label">Location-specific need</div>
        </div>
        <span class="subtitle">Lower numbers are higher priority</span>
        <hr/>
        <h5>Categories</h5>
        {#each categories as cat}
            <div class="legend-item">
                <div class="marker marker-category" style="background-color: {cat.color}">{cat.letter}</div>
                <div class="label">{cat.name}</div>
            </div>
        {/each}
        <span class="subtitle">Click on points or lines for more details</span>
    </div>
</div>

<style>
    .map-view {
        position: relative;
    }

    #map {
        width: 100%;
        height: 700px;
    }

    .legend {
        line-height: 18px;
        color: #555;
        position: absolute;
        bottom: 40px;
        right: 0px;
        z-index: 2;
        background-color: rgba(255, 255, 255, 0.7);
        padding: 5px 10px;
    }

    .legend h4, h5 {
        margin: 5px 0px;
    }

    .legend .subtitle{
        font-style: italic;
        font-size: 0.9rem;
    }

    .legend hr {
        background-color: #c1c1c1;
        color: #c1c1c1;
        margin: 6px 2px;
    }

    .legend-item {
        letter-spacing: -0.2px;
        margin-right: 1rem;
        display: flex;
        flex-direction: row;
    }

    .legend-item .marker {
        padding-right: 1px;
        padding-bottom: 1px;
        cursor: default;
    }

    .label {
        margin-left: 1rem;
    }

    :global(.marker) {
        text-align: center;
        background-color: #363636;
        color: white;
        font-weight: 600;
        line-height: 15.5px;
        height: 15px;
        width: 15px;
        font-size: 11px;
        border: 1px solid rgba(255, 255, 255, 0.6);
        cursor: pointer;
    }

    :global(.marker-category) {
        background-color: #9e9e9e;
    }

    :global(.marker-district) {
        border-radius: 50%;

    }

    :global(.marker-location) {
        border-radius: 1px;
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

    :global(.mapboxgl-popup-content p) {
        margin: 4px 0px;
    }

</style>