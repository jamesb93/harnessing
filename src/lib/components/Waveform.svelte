<script>
    import { onMount } from "svelte";
    import Container from '$lib/components/Container.svelte';
    import Button from '$lib/components/Button.svelte';
    import { noext, cloudPrefix } from '$lib/utility/paths.js';

    export let segments;
    export let points;
    export let title = "title";
    export let caption = "";
    export let file;
    export let peaks;
    export let id = "";

    const noExtension = noext(file);
    const lossless = cloudPrefix + noExtension + '.wav'

    // Form path to lossless download
    let Peaks, instance, overview, zoom, audio, controls, peaksControls;
    let lastSelected = 0;

    const convert = (time) => {
        const date = new Date(time * 1000).toISOString().substr(11, 8)
        return date.toString().substr(3);
    }

    onMount (async()=>{
        const module = await import("peaks.js");
        Peaks = module.default;
        const options = {
            containers: {
                zoomview: zoom,
                overview: overview
            },
            dataUri: { 
                arraybuffer: peaks 
            },
            mediaElement: audio,
            zoomWaveformColor: 'rgba(0, 30, 128, 0.61)',
            overviewWaveformColor: 'rgba(0, 15, 100, 0.3)',
            overviewHighlightColor: 'grey',
            playheadColor: 'rgba(0, 0, 0, 1)',
            playheadTextColor: '#aaa',
            showPlayheadTime: false,
            pointMarkerColor: '#FF0000',
            axisGridlineColor: '#ccc',
            axisLabelColor: '#aaa',
            randomizeSegmentColor: true,
            segments: segments,
            points: points
        }
        instance = Peaks.init(options, (err, p) => {
            if (err) {
                console.log(err)
            } else {
                instance = p
                instance.views.getView('overview').fitToContainer();
            }
        });
    });

    function clickHandler(segment, i) {
        instance.player.seek(segment.startTime)
        lastSelected = i
    }
</script>

<Container id={id}>
    <div class="horizontal">
        <span id="title">{title}</span>
        <span id="caption">{caption}</span>
    </div>

    <div class="vis">
        <div id='waveform-overview' bind:this={overview} />
        <div id='waveform-zoom' bind:this={zoom} />
    </div>

    <div class="peaks-controls" bind:this={peaksControls}>
        <audio controls bind:this={audio}>
            <source src={file} type="audio/mp3">
            <track kind="captions">
        </audio>
        <div bind:this={controls} class="audio-controls">
            <Button clickHandler={ () => instance.zoom.zoomIn() } text="+" />
            <Button clickHandler={ () => instance.zoom.zoomOut() } text="-" />
        </div>
    </div>
    {#if segments}
    <ul class="segments">
        <span id="timecodes">List of referenced time codes</span>
        {#each segments as segment, i}
            <div class:selected={ lastSelected === i } class='timecode' on:click={ () => clickHandler(segment, i) }>
                <span id='time'>{convert(segment.startTime)} - {convert(segment.endTime)}</span>
                <span id="label">{segment.labelText}</span>
            </div>
        {/each}
    </ul>
    {/if}

    <div id='lossless'>
        <a rel='external' id='lossless-link' href={lossless} download>
            Download Lossless Version
        </a>
    </div>
</Container>

<style>

    #waveform-overview {
        height: 65px;
    }

    #lossless {
        margin-top: 10px;
    }
    #lossless-link {
        font-style: italic;
        text-align: left;
        color: rgb(96, 96, 96);
    }

    #lossless-link:hover {
        background-color: var(--dark-blue);
        color: white;
    }
    
    .audio-controls {
        display: flex;
        flex-direction: row;
    }
    
    #title {
        text-align: left;
		font-weight: bold;
    }

    .timecode {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        word-wrap: none;
        border-top: 1px rgb(197, 197, 197) solid;
        gap: 3%;
    }

    .timecode:hover {
        background-color: rgb(240, 240, 240);
    }

    .timecode:active {
        background-color: rgb(199, 199, 199);
    }

    #time {
        white-space: nowrap;
    }

    #label {
        color: grey;
        text-align: right;
    }
    
    #caption{
        font-style: italic;
        min-width: max-content;
    }
    
    .horizontal {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        gap: 10%;
        padding-bottom: 15px;
    }
    
    .vis {
        padding-bottom: 5px;
        display: flex;
        flex-direction: column;
        gap: 5px;
    }
    
    .peaks-controls {
        display: flex;
        justify-content: space-between;
        padding-top: 3px;
        align-items: center;
    }

    a { 
        color: black
    }

    a:hover {
        background-color: inherit;
        text-decoration: none;
    }

    .selected {
        font-weight: bold;
    }

    .segments {
        padding-top: 5px;
        margin-top: 1em;
        padding-left: 1em;
        padding-right: 1em;
        border-top: 1px rgb(197, 197, 197) solid;
    }
</style>


    