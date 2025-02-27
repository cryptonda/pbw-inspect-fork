<script>
    import { page } from '$app/stores';
    import { config } from '$lib/pbw';
    //import { redirect } from '@sveltejs/kit';
	import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import SvelteMarkdown from 'svelte-markdown';
    import Footer from '$lib/components/Footer.svelte';
    import EventTypeBadge from '$lib/components/EventTypeBadge.svelte';
    import Timeline from "$lib/components/Timeline.svelte";
    import { formatItemDate, bareDomain, getFlagEmoji } from '$lib/utils.js';
    import makeBlockie from 'ethereum-blockies-base64';
	import TimelineItem from '../../../lib/components/TimelineItem.svelte';

    export let data;

    let entry = $page.params.entry
    $: type = $page.params.type;
    $: tc = config.collections[type]
    $: items = data.bundle[type]


    function processItems(_items) {
        if (!_items) return [];
        _items = JSON.parse(JSON.stringify(_items))
        if (type === 'events') {
            _items = _items.sort((x, y) => x.date > y.date ? 1 : -1)
        }
        return _items
    }

    $: processedItems = processItems(items)

    onMount(async () => {
        if (!config.collections[$page.params.type]) {
            const ftype = Object.keys(config.collections).find(k => config.collections[k].model === $page.params.type)
            if (ftype) {
                goto(`/${$page.params.entry}/${ftype}`)
            }
        }
    })

</script>

<svelte:head>
    <title>{tc?.title} | #PBW{$page.params.entry} Inspector</title>
</svelte:head>

{#if tc}
<div class="w-full">
    <div class="max-w-7xl mx-auto pt-5 md:pt-10">
        <div class="mx-4 xl:mx-0">
            <div class="flex gap-8 mb-6 md:mb-10">
                <h1 class="text-4xl md:text-5xl font-bold text-pbw-red"><a href="/{$page.params.entry}">#PBW23</a><span class="text-pbw-yellow">.{$page.params.type}</span></h1>
            </div>
            <div class="flex flex-wrap md:flex-nowrap w-full">
            </div>
            <h2 class="text-2xl uppercase font-bold text-gray-500">{tc.title} ({processedItems.length})</h2>
            <div class="text-xl mt-6 text-gray-500 dark:text-gray-400">  
                <table class="w-full table-auto">
                    <thead>
                        <tr class="text-left">
                            {#if type === 'events'}
                                <th class="text-right pr-4">📅</th>
                                <th></th>
                                <th>Name</th>
                                <th class="hidden md:table-cell">📍</th>
                                <th class="hidden md:table-cell">👥</th>
                            {/if}
                            {#if type === 'speakers'}
                                <th></th>
                                <th>Name</th>
                                <th>🌎</th>
                                <th class="hidden md:table-cell">🐦</th>
                                <th class="hidden md:table-cell">Bio</th>
                            {/if}
                            {#if type === 'media-partners'}
                                <th></th>
                                <th>Name</th>
                                <th>🌎</th>
                                <th class="hidden md:table-cell">Description</th>
                            {/if}
                            {#if type === 'benefits'}
                                <th></th>
                                <th>Name</th>
                            {/if}
                            {#if type === 'places'}
                                <th></th>
                                <th>Name</th>
                                <th>👥</th>
                                <th class="hidden md:table-cell">Address</th>
                            {/if}
                        </tr>
                    </thead>
                    <tbody>
                        {#each processedItems as item}
                            <tr class="hover:bg-pbw-yellow/20 dark:hover:bg-pbw-white/10">
                                {#if type === 'events'}
                                    <td class="text-right pr-2 md:pr-4 text-base md:text-xl">{formatItemDate(item)}</td>
                                    <td class="w-12 md:w-14">
                                        <a href="/{entry}/{tc.model}/{item.id}">
                                            {#if item.logo}
                                                <img src={item.logo} class="w-10 rounded aspect-square object-cover" />
                                            {:else if item.types && item.types[0]}
                                                <!--div class="w-10 h-10 pt-2 rounded" style="background-color: {config.eventTypeColors[item.types[0]]};"></div-->
                                                <div class="w-10 h-10 pt-2 rounded" style="background: url({makeBlockie('0x'+item.hash.substr(0,40))}); background-size: 100% 100%;"></div>
                                            {/if}
                                        </a>
                                    </td>
                                    <td class="text-lg md:text-2xl flex items-center h-12">
                                        <div class=""><a href="/{entry}/{tc.model}/{item.id}" class="text-pbw-red hover:underline">{item.name}</a></div>
                                        <div class="gap-1 items-center ml-4 hidden md:flex">
                                            {#each item.types as type}
                                                <EventTypeBadge {type} />
                                            {/each}
                                        </div>
                                    </td>
                                    <td class="hidden md:table-cell">
                                        {#if item.venues}
                                            {@html item.venues.map(vId => {
                                                const place = data.bundle.places.find(p => p.id === vId)
                                                return `<a href="/${$page.params.entry}/place/${place.id}" class=\"underline hover:no-underline\">${place.name}</a>`
                                            }).join(", ")}
                                        {:else if item.venueName}
                                            {#if item.venueUrl}
                                                <a href={item.venueUrl} class="underline hover:no-underline external">{item.venueName}</a>
                                            {:else}
                                                {#if item.venueName && item.venueName !== "TBA"}{item.venueName}{:else}<span class="opacity-50">TBA</span>{/if}
                                            {/if}
                                        {/if}
                                    </td>
                                    <td class="hidden md:table-cell">{#if item.attendees}{item.attendees}{:else}<span class="opacity-50">TBA</span>{/if}</td>
                                    <td class="hidden md:table-cell">
                                        {#if item.languages && item.languages.length > 0}
                                            <div class="flex gap-1">
                                                {#each item.languages as lang}
                                                    <div>{getFlagEmoji(lang)}</div>
                                                {/each}
                                            </div>
                                        {/if}
                                    </td>
                                {/if}
                                {#if type === 'speakers'}
                                    <td class="w-12 md:w-14">
                                        <img src={item.photoUrl} class="w-10 inline-block rounded aspect-square object-cover" />
                                    </td>
                                    <td class="text-2xl h-12">
                                        <a href="/{entry}/{tc.model}/{item.id}" class="text-pbw-red hover:underline">{item.name}</a>
                                    </td>
                                    <td>{item.country && item.country !== 'xx' ? getFlagEmoji(item.country, false) : ''}</td>
                                    <td class="hidden md:table-cell">
                                        {#if item.twitter}
                                            @<a href="https://twitter.com/{item.twitter}" class="underline hover:no-underline">{item.twitter}</a>
                                        {/if}
                                    </td>
                                    <td class="hidden md:table-cell"><SvelteMarkdown source={item.caption} /></td>
                                {/if}
                                {#if type === 'media-partners'}
                                    <td class="w-20">
                                        <img src={item.logo} class="w-16 inline-block rounded aspect-[16/9] object-cover" />
                                    </td>
                                    <td class="text-2xl h-12">
                                        <a href="/{entry}/{tc.model}/{item.id}" class="text-pbw-red hover:underline">{item.name}</a>
                                    </td>
                                    <td>
                                        {#if item.languages && item.languages.length > 0}
                                            <div class="flex gap-1">
                                                {#each item.languages as lang}
                                                    <div>{getFlagEmoji(lang)}</div>
                                                {/each}
                                            </div>
                                        {/if}
                                    </td>
                                    <td class="hidden md:table-cell">
                                        <SvelteMarkdown source={item.description} />
                                    </td>   
                                {/if}    
                                {#if type === 'benefits'}
                                    <td class="w-14">
                                        <img src={item.logo} class="w-10 inline-block rounded aspect-square object-cover" />
                                    </td>
                                    <td class="text-2xl h-12">
                                        <a href="/{entry}/{tc.model}/{item.id}" class="text-pbw-red hover:underline">{item.name}</a>
                                    </td>
                                {/if}                 
                                {#if type === 'places'}
                                    <td class="w-14">
                                        <img src={item.photo} class="w-10 inline-block rounded aspect-square object-cover" />
                                    </td>
                                    <td class="text-2xl h-12">
                                        <a href="/{entry}/{tc.model}/{item.id}" class="text-pbw-red hover:underline">{item.name}</a>
                                    </td>
                                    <td>{item.capacity}</td>
                                    <td class="hidden md:table-cell">{item.address}</td>
                                {/if}                  
                            </tr>
                        {/each}
                    </tbody>
                </table>
            </div>
            <Footer bundle={data.bundle} />
        </div>
    </div>
</div>
{/if}
{#if type === 'timeline'} 
<div class="w-full">
    <div class="max-w-7xl mx-auto pt-5 md:pt-10">
        <div class="mx-4 xl:mx-0">
            <div class="flex gap-8 mb-6 md:mb-10">
                <h1 class="text-4xl md:text-5xl font-bold text-pbw-red"><a href="/{$page.params.entry}">#PBW23</a><span class="text-pbw-yellow">.{$page.params.type}</span></h1>
            </div>
        </div>
    </div>
</div>

    <Timeline />
    <div class="bg-sky-800 h-[1000px]">high content below to test sticky header</div>
{/if}

