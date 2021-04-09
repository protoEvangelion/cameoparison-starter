<script>
    import { createEventDispatcher } from "svelte";
    import Card from "../components/Card.svelte";
    import { sleep, pick_random } from "../utils";
    export let selection;

    const dispatch = createEventDispatcher();

    const loadDetails = async (celeb) => {
        const res = await fetch(
            `https://cameo-explorer.netlify.app/celebs/${celeb.id}.json`
        );

        return await res.json();
    };

    const promises = selection.map((round) =>
        Promise.all([loadDetails(round.a), loadDetails(round.b)])
    );

    const results = Array(selection.length);

    let i = 0;

    let lastResult;
    let done;
    const pickMessage = (p) => {
        if (p < 0.5)
            return pick_random(["Ouch", "That was not good", "Must try again"]);
        if (p <= 0.8)
            return pick_random([
                "Not bad keep practicing",
                "Do you even culture bro?",
            ]);
        if (p <= 1) return pick_random(["So close!", "Almost there!"]);

        return pick_random(["You rock!", "Wahoo!"]);
    };
    const submit = async (a, b, sign) => {
        lastResult = Math.sign(a.price - b.price) === sign ? "right" : "wrong";

        // await sleep(1500);
        await sleep(1);
        results[i] = lastResult;
        lastResult = null;
        if (i < selection.length - 1) {
            i += 1;
        } else {
            done = true;
        }
    };

    $: score = results.filter((x) => x === "right").length;
</script>

<header>
    <p>
        Tap on the more monetisable celebrity's face, or tap 'same price' if
        society values them equally.
    </p>
</header>

<div class="game-container">
    {#if done}
        <div class="done">
            <strong>{score}/{results.length}</strong>
            <p>{pickMessage(score / results.length)}</p>
            <button on:click={() => dispatch("restart")}>Back to Home</button>
        </div>
    {:else}
        {#await promises[i] then [a, b]}
            <div class="game">
                <Card celeb={a} on:select={() => submit(a, b, 1)} />

                <button class="same" on:click={() => submit(a, b, 0)}
                    >same price</button
                >

                <Card celeb={b} on:select={() => submit(a, b, -1)} />
            </div>
        {:catch}
            <p class="error">failed to load data</p>
        {/await}
    {/if}
</div>

<div class="results">
    {#each results as result}
        <span class="result">
            {#if result}
                <img src="/icons/{result}.svg" alt="" />
            {/if}
        </span>
    {/each}
</div>

{#if lastResult}
    <img class="giant-result" src="/icons/{lastResult}.svg" alt="" />
{/if}

<style>
    .giant-result {
        position: fixed;
        width: 50vmin;
        height: 50vmin;
        left: calc(50vw - 25vmin);
        top: calc(50vh - 25vmin);
        opacity: 0.5;
    }
    .game {
        display: flex;
        align-items: center;
        grid-template-rows: 1fr 2em 1fr;
        grid-gap: 0.5em;
        width: 100%;
        height: 100%;
        max-width: min(100%);
        margin: 0 auto;
    }

    .game-container {
        flex: 1;
    }
    .error {
        color: red;
    }
    .same {
        min-width: 125px;
    }

    @media (min-width: 640px) {
        .game {
            max-width: 100%;
            grid-template-rows: none;
            grid-template-columns: 1fr 8em 1fr;

            max-height: calc(100vh - 6em);
        }

        .same {
            height: 8em;
        }
    }
</style>
