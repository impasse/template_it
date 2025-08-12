<script>
    import { Liquid } from "liquidjs";

    const liquid = new Liquid();

    let template = $state(localStorage["template"] || "");
    let jsonObject = $state(JSON.parse(localStorage["jsonObject"] || "{}"));

    $effect(() => {
        localStorage["template"] = template;
        localStorage["jsonObject"] = JSON.stringify(jsonObject);
    });
</script>

<svelte:head>
    <title>Liquid Template Renderer</title>
    <meta
        name="description"
        content="A simple Liquid template renderer with Svelte"
    />
</svelte:head>

<div class="grid grid-cols-2 grid-rows-2 p-4 w-full h-screen gap-3">
    <div class="row-span-2 flex flex-col">
        <h1 class="font-bold">liquid template</h1>
        <textarea
            class="flex-1 outline w-full h-full resize-none p-1"
            bind:value={template}
        ></textarea>
    </div>
    <div class="flex flex-col">
        <h2 class="font-bold">Object</h2>
        <textarea
            class="flex-1 w-full h-full outline resize-none p-1"
            bind:value={
                () => JSON.stringify(jsonObject, null, 4),
                (v) => (jsonObject = JSON.parse(v))
            }
        ></textarea>
    </div>
    <div class="flex flex-col">
        <h2 class="font-bold">Rendered</h2>
        {#await liquid.parseAndRender(template, jsonObject)}
            <div class="flex-1 w-full h-full outline p-1">Loading...</div>
        {:then result}
            <div class="flex-1 w-full h-full outline overflow-auto p-1">
                {result}
            </div>
        {:catch e}
            <div class="flex-1 w-full h-full outline text-red-500 p-1">
                {e.message}
            </div>
        {/await}
    </div>
</div>
