<script>
    import { onMount } from "svelte";

    let id = undefined;

    onMount(() => {
        window.Twitch.ext.onAuthorized(function () {
            const config = window.Twitch.ext.configuration.broadcaster.content;
            id = new URL(config).pathname.split("/")[2];
        });
    });

    async function getLoadouts() {
        const res = await fetch("https://karl.gg/graphql", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                query: `
                {
                    myLoadouts(userId: ${id}) {
                        id
                        name
                        updated_at
                        character {
                            id
                        }
                    }
                }`,
            }),
        });
        const loadouts = (await res.json()).data.myLoadouts;
        loadouts.sort((a, b) => (a.updated_at < b.updated_at ? 1 : -1));
        return loadouts;
    }
</script>

<div class="header">
    <h1>My Loadouts</h1>
    <a href="https://karl.gg" target="_blank">
        <img src="./logo.svg" alt="karl.gg icon" />
    </a>
</div>

{#if id != undefined}
    {#await getLoadouts()}
        <svg
            class="animate-spin -ml-1 mr-3 h-10 w-10 text-white"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
        >
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
            <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
        </svg>
    {:then loadouts}
        <div class="list">
            {#each loadouts as loadout}
                <a class="loadout" href={`https://karl.gg/preview/${loadout.id}/`} target="_blank">
                    <img src={`./class_${loadout.character.id}.png`} alt={`class ${loadout.character.id} icon`} />
                    {loadout.name}
                </a>
            {/each}
        </div>
    {/await}
{:else}
    Error
{/if}

<style lang="postcss">
    .list {
        @apply flex flex-col gap-2;
    }
    .loadout {
        @apply flex gap-2 p-2 text-lg;
        @apply bg-gray-700 hover:bg-gray-800;
        @apply last:mb-4;
    }

    .loadout img {
        @apply self-start;
    }
</style>
