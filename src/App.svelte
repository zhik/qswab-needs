<script>
    import {onMount} from 'svelte'
    import {needsData} from './stores'
    import MapView from './components/MapView.svelte'

    onMount(() => {
        (async () => {
            // Load data from needs.geojson
            const features = await (await fetch('data/needs.geojson')).json()
            needsData.set(features)
        })();
    })
</script>

<main>
    <MapView/>
</main>

<style>
    main {
        padding: 1em 0.5em;
        margin: 0 auto;
    }

    @media (max-width: 768px) {
        main {
            padding: 1em 2em;
        }
    }
</style>