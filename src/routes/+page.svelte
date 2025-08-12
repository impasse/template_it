<script>
    import { Liquid } from "liquidjs";

    const liquid = new Liquid();

    let template = $state(localStorage['template'] || '');
    let jsonObject = $state(JSON.parse(localStorage['jsonObject'] || '{}'));

    $effect(() => {
        localStorage['template'] = template;
        localStorage['jsonObject'] = JSON.stringify(jsonObject);
    });
</script>

<svelte:head>
    <title>Liquid Template Renderer</title>
    <meta name="description" content="A simple Liquid template renderer with Svelte">
</svelte:head>

<div class="grid grid-cols-2 gap-x-2 p-8 min-h-screen">
    <div>
        <h1 class="font-bold">liquid template</h1>
        <textarea class="outline w-full h-full resize-none" bind:value={template}
        ></textarea>
    </div>
    <div class="grid grid-cols-1 grid-rows-2 gap-y-8">
        <div>
            <h2 class="font-bold">Object</h2>
            <textarea
                class="w-full h-full outline resize-none"
                bind:value={
                    () => JSON.stringify(jsonObject, null, 4),
                    (v) => (jsonObject = JSON.parse(v))
                }
            ></textarea>
        </div>
        <div>
            <h2 class="font-bold">Rendered</h2>
            {#await liquid.parseAndRender(template, jsonObject)}
                <div class="w-full h-full outline">Loading...</div>
            {:then result}
                <div class="w-full h-full outline">{result}</div>
            {:catch e}
                <div class="w-full h-full outline text-red-500">{e.message}</div>
            {/await}
        </div>
    </div>
</div>
