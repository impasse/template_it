<script>
    import { Liquid } from "liquidjs";

    const liquid = new Liquid();

    const MAX_HISTORY = 10;
    const STORAGE_KEY = "liquidTemplateHistory";

    let template = $state("");
    let jsonObject = $state({});
    let jsonText = $state("");
    let title = $state("");
    let history = $state([]);
    let currentHistoryIndex = $state(-1);

    function loadHistory() {
        const saved = localStorage.getItem(STORAGE_KEY);
        if (saved) {
            const parsedHistory = JSON.parse(saved);
            history = parsedHistory.map(item => ({
                template: item.template || "",
                title: item.title || "",
                jsonObject: item.jsonObject || {},
                timestamp: item.timestamp || new Date().toISOString()
            }));
            if (history.length > 0) {
                loadHistoryItem(history.length - 1);
            }
        }
    }

    function saveHistory() {
        if (!template.trim()) {
            showSaveMessage("Cannot save empty template");
            return;
        }
        
        const historyItem = {
            template: template,
            title: title.trim() || undefined,
            jsonObject: jsonObject,
            timestamp: new Date().toISOString()
        };

        if (currentHistoryIndex >= 0 && currentHistoryIndex < history.length) {
            const current = history[currentHistoryIndex];
            if (current.template === template && 
                JSON.stringify(current.jsonObject) === JSON.stringify(jsonObject) &&
                current.title === title.trim()) {
                showSaveMessage("No changes to save");
                return;
            }
        }

        history = [...history.slice(-MAX_HISTORY + 1), historyItem];
        currentHistoryIndex = history.length - 1;
        
        localStorage.setItem(STORAGE_KEY, JSON.stringify(history));
        showSaveMessage("Saved successfully");
    }

    function loadHistoryItem(index) {
        if (index >= 0 && index < history.length) {
            const item = history[index];
            template = item.template;
            jsonObject = item.jsonObject;
            title = item.title || "";
            jsonText = JSON.stringify(item.jsonObject, null, 2);
            currentHistoryIndex = index;
            showSaveMessage("Loaded successfully");
        }
    }

    function deleteHistoryItem(index) {
        history = history.filter((_, i) => i !== index);
        if (currentHistoryIndex >= index) {
            currentHistoryIndex = Math.max(0, currentHistoryIndex - 1);
        }
        if (history.length > 0 && currentHistoryIndex < history.length) {
            loadHistoryItem(currentHistoryIndex);
        } else {
            template = "";
            jsonObject = {};
            currentHistoryIndex = -1;
        }
        localStorage.setItem(STORAGE_KEY, JSON.stringify(history));
        showSaveMessage("Deleted successfully");
    }

    function clearHistory() {
        history = [];
        currentHistoryIndex = -1;
        template = "";
        jsonObject = {};
        title = "";
        localStorage.removeItem(STORAGE_KEY);
        showSaveMessage("History cleared");
    }

    let saveMessage = $state("");
    let isDirty = $state(false);

    function showSaveMessage(message) {
        saveMessage = message;
        setTimeout(() => {
            saveMessage = "";
        }, 2000);
    }

    
    loadHistory();

    $effect(() => {
        try {
            if (jsonText.trim()) {
                const parsed = JSON.parse(jsonText);
                jsonObject = parsed;
            }
        } catch (err) {
            // Invalid JSON, keep the current jsonObject
        }
    });

    $effect(() => {
        if (currentHistoryIndex >= 0 && currentHistoryIndex < history.length) {
            const current = history[currentHistoryIndex];
            const isChanged = current.template !== template || 
                            JSON.stringify(current.jsonObject) !== JSON.stringify(jsonObject) ||
                            current.title !== title.trim();
            isDirty = isChanged;
        } else {
            isDirty = template.trim() !== "" || Object.keys(jsonObject).length > 0 || title.trim() !== "";
        }
    });
</script>

<svelte:head>
    <title>Liquid Template Renderer</title>
    <meta
        name="description"
        content="A simple Liquid template renderer with Svelte"
    />
</svelte:head>

<div class="flex flex-col h-screen">
    <header class="bg-white border-b p-4">
        <div class="flex items-center justify-between">
            <h1 class="text-2xl font-bold text-gray-800">Liquid Template Renderer</h1>
            <div class="flex items-center gap-3">
                {#if saveMessage}
                    <div class="px-3 py-1 bg-green-100 text-green-800 rounded text-sm">
                        {saveMessage}
                    </div>
                {/if}
                {#if isDirty}
                    <div class="px-2 py-1 bg-yellow-100 text-yellow-800 rounded text-xs">
                        Unsaved changes
                    </div>
                {/if}
                <button 
                    class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2"
                    onclick={saveHistory}
                    disabled={!template.trim()}
                >
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3 3m0 0l-3-3m3 3V4"></path>
                    </svg>
                    Save
                </button>
            </div>
        </div>
    </header>

    <div class="flex-1 grid grid-cols-1 lg:grid-cols-3 gap-4 p-4">
        <div class="lg:col-span-2 flex flex-col space-y-4">
            <div class="flex-1 flex flex-col bg-white rounded-lg shadow-sm border">
                <div class="p-3 border-b flex justify-between items-center">
                    <h2 class="font-semibold text-gray-700">Template</h2>
                    <input 
                        type="text" 
                        class="px-3 py-1 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent" 
                        placeholder="Enter title (optional)..."
                        bind:value={title}
                    >
                </div>
                <textarea
                    class="flex-1 p-3 border-0 resize-none focus:outline-none font-mono text-sm"
                    bind:value={template}
                    placeholder="Enter your Liquid template here..."
                ></textarea>
            </div>

            <div class="flex-1 grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="flex flex-col bg-white rounded-lg shadow-sm border">
                    <div class="p-3 border-b">
                        <h2 class="font-semibold text-gray-700">JSON Data</h2>
                    </div>
                    <textarea
                        class="flex-1 p-3 border-0 resize-none focus:outline-none font-mono text-sm"
                        bind:value={jsonText}
                    ></textarea>
                </div>

                <div class="flex flex-col bg-white rounded-lg shadow-sm border">
                    <div class="p-3 border-b">
                        <h2 class="font-semibold text-gray-700">Rendered Output</h2>
                    </div>
                    {#await liquid.parseAndRender(template, jsonObject)}
                        <div class="flex-1 p-3 text-gray-500">Loading...</div>
                    {:then result}
                        <div class="flex-1 p-3 border-0 overflow-auto font-mono text-sm whitespace-pre-wrap">
                            {result}
                        </div>
                    {:catch e}
                        <div class="flex-1 p-3 text-red-500 font-mono text-sm">
                            Error: {e.message}
                        </div>
                    {/await}
                </div>
            </div>
        </div>

        <div class="flex flex-col bg-white rounded-lg shadow-sm border">
            <div class="p-3 border-b flex justify-between items-center">
                <h2 class="font-semibold text-gray-700">History</h2>
                <div class="flex items-center gap-2">
                    <span class="text-xs text-gray-500">{history.length}/{MAX_HISTORY}</span>
                    {#if history.length > 0}
                        <button 
                            class="px-2 py-1 bg-red-500 text-white rounded hover:bg-red-600 text-xs"
                            onclick={clearHistory}
                        >
                            Clear
                        </button>
                    {/if}
                </div>
            </div>
            <div class="flex-1 overflow-auto">
                {#if history.length === 0}
                    <div class="p-4 text-gray-500 text-sm text-center">
                        <svg class="w-8 h-8 mx-auto mb-2 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                        </svg>
                        No saved templates yet
                    </div>
                {:else}
                    <div class="divide-y">
                        {#each history as item, index}
                            <div 
                                class="p-3 hover:bg-gray-50 cursor-pointer transition-colors {currentHistoryIndex === index ? 'bg-blue-50 border-l-4 border-blue-500' : ''}"
                                role="button"
                                tabindex="0"
                                onclick={() => loadHistoryItem(index)}
                                onkeydown={(e) => e.key === 'Enter' && loadHistoryItem(index)}
                            >
                                <div class="flex justify-between items-start">
                                    <div class="flex-1 min-w-0">
                                        <div class="text-sm font-medium text-gray-900 truncate">
                                            {item.title ? item.title : (item.template ? item.template.slice(0, 60) + (item.template.length > 60 ? '...' : '') : 'Empty template')}
                                        </div>
                                        <div class="text-xs text-gray-500 mt-1">
                                            {new Date(item.timestamp).toLocaleString()}
                                        </div>
                                        {#if !item.title && item.template}
                                            <div class="text-xs text-gray-400 mt-1">
                                                {Object.keys(item.jsonObject).length} keys
                                            </div>
                                        {/if}
                                    </div>
                                    <button 
                                        class="ml-2 p-1 text-red-500 hover:bg-red-50 rounded transition-colors"
                                        onclick={(e) => {
                                            e.stopPropagation();
                                            deleteHistoryItem(index);
                                        }}
                                        title="Delete"
                                        aria-label="Delete"
                                    >
                                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        {/each}
                    </div>
                {/if}
            </div>
        </div>
    </div>
</div>
