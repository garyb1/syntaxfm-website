<script lang="ts">
	import Icon, { type IconName } from "./Icon.svelte";
    import type { LatestShow } from '$server/ai/queries';
	import { PLAYER_STATUS, player } from '$state/player';
	import { capitalize } from "$/utilities/capitalize";
	import type { Show } from "@prisma/client";

    export let show: Show | null;
    export let hideLabel: boolean = false;
	export let display: 'list' | 'card' | 'highlight' | '' = '';
    export let showEpisodeLabel = false;

    let isPlaying: boolean;
    let playerState: Extract<IconName, 'play' | 'playing'>;
    let isCurrentlyPlayingShow: boolean;
    let episodeLabel = showEpisodeLabel ? 'Episode' : '';

    $: isCurrentlyPlayingShow = $player.current_show?.number === show?.number;
	$: isPlaying = isCurrentlyPlayingShow && $player.status === PLAYER_STATUS.PLAYING;
    $: playerState = isPlaying ? 'playing' : 'play';

	function play_show() {
        if (show) {
            const currentTime = isCurrentlyPlayingShow ? $player.currentTime : 0;
            player.start_show(show, currentTime);
        }
	}
</script>

<button 
    data-testid="play-show" 
    on:click|preventDefault={play_show} 
    class:highlight={display === 'highlight'} 
    class:list={display === 'list'}
    aria-labelledby="{show?.id}-detail {show?.id}-title"
    aria-live="polite"
    aria-atomic="true"
>
    <Icon name={playerState} />
    <span class:visually-hidden={hideLabel} id="{show?.id}-detail">
        {capitalize(playerState)} {episodeLabel} #{show?.number}
    </span>
</button>

<style lang="postcss">
    button:focus {
        outline: 2px solid var(--fg);
        outline-offset: 2px;
    }

    .list {
        background: transparent;
		border-radius: 50%;
		align-self: center;
		border-width: 1px;
		padding: 10px;
		box-shadow: inset 0 0 0 2px color-mix(in lch, var(--fg) 50%, transparent 94%);
		color: var(--fg);
		outline: 2px solid transparent;
	}
</style>