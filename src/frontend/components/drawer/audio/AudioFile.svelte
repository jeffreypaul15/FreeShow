<script lang="ts">
    import { activeFocus, activeShow, focusMode, media, outLocked, playingAudio } from "../../../stores"
    import Icon from "../../helpers/Icon.svelte"
    import { getAudioDuration, playAudio, startPlaylist } from "../../helpers/audio"
    import { joinTime, secondsToTime } from "../../helpers/time"
    import Button from "../../inputs/Button.svelte"

    import SelectElem from "../../system/SelectElem.svelte"

    export let path: string
    export let name: string
    export let active: string | null = null
    export let playlist: boolean = false
    export let index: number = -1
    export let fileOver: boolean = false
</script>

<SelectElem id="audio" data={{ path, name, index }} {fileOver} borders={playlist ? "edges" : "all"} trigger={playlist ? "column" : null} draggable>
    <Button
        class="context #audio_button{playlist ? '_playlist' : ''}"
        outline={$playingAudio[path]}
        active={$activeShow?.id === path}
        border
        style="width: 100%;"
        title={path}
        bold={false}
        on:click={(e) => {
            if ($outLocked || e.ctrlKey || e.metaKey) return

            if (playlist) startPlaylist(active, path, true)
            else playAudio({ path, name }, true, 0, e.altKey)
        }}
        on:dblclick={(e) => {
            if (e.ctrlKey || e.metaKey) return

            if ($focusMode) activeFocus.set({ id: path })
            else activeShow.set({ id: path, name, type: "audio" })
        }}
    >
        <span style="max-width: 90%;">
            <Icon
                id={$playingAudio[path]?.paused === true ? "play" : $playingAudio[path]?.paused === false ? "pause" : $media[path]?.favourite === true && active !== "favourites" ? "star" : "music"}
                white={$playingAudio[path]?.paused === true || (!$playingAudio[path] && ($media[path]?.favourite !== true || active === "favourites"))}
                right
            />
            <p>{name.slice(0, name.lastIndexOf("."))}</p>
        </span>
        <span style="opacity: 0.8;">
            {#await getAudioDuration(path)}
                <p>00:00</p>
            {:then duration}
                <p>{joinTime(secondsToTime(duration))}</p>
            {/await}
        </span>
    </Button>
</SelectElem>

<style>
    span {
        display: flex;
        align-items: center;
        gap: 5px;
    }
</style>
