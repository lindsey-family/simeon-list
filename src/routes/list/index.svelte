<script>
    import { MetaTags } from "svelte-meta-tags";
    import Button from "../../lib/components/Button.svelte";
    import { page } from '$app/stores'
    const sort = $page.url.searchParams.get('sort')
    export let items
    async function updateItem(e) {
        var msg = document.getElementById(`msg-${e.target.id.value}`)
        msg.innerHTML = 'updating...'
        try {
            const formData = new FormData(e.target)
            const parsed = await parseFormData(formData);
            const data = {}
            const checked = parsed[`checkbox-${parsed.id}`]
            if (checked == 'on') {
                data.checked = true
            } else {
                data.checked = false
            }
            data.id = parsed.id
            const res = await updateReq(data)
            msg.innerText = 'updated'
            setTimeout(() => {
                msg.innerText = ""
                setTimeout(() => {
                    window.location.reload()
                }, 100)
            }, 400);
        } catch (error) {
            msg.innerText = `couldn't update :(`
            setTimeout(() => {
                msg.innerText = ""
            }, 1000);
        }
    }
    async function parseFormData(formData) {
        const data = {};
        for (let field of formData) {
            const [key, value] = field;
            data[key] = value;
        }
        return data
    }
    async function updateReq(data) {
        const res = await fetch('/update', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        })
        const json = await res.json()
        return json
    }
</script>
<h1 class="text-3xl dark:text-white text-black">Simeon's List</h1>
{#if items.length > 1}
<i class="text-darkgrey dark:text-grey">{items.length} items</i>
{:else if items.length == 1}
<i class="text-darkgrey dark:text-grey">1 item</i>
{:else if items.length == 0}
<i class="text-darkgrey dark:text-grey">no items yet :(</i>
{/if}
<form>
    <label for="sort" class="text-black dark:text-grey">Sort by:</label>
    <select name="sort" id="sort" class="dark:bg-grey">
        {#if sort == null || 'all'}
        <option value="all" selected>All</option>
        {:else}
        <option value="all">All</option>
        {/if}
        {#if sort == 'ranking-asc'}
        <option value="ranking-asc" selected>Priority (Low to High)</option>
        {:else}
        <option value="ranking-asc">Priority (Low to High)</option>
        {/if}
        {#if sort == 'ranking-desc'}
        <option value="ranking-desc" selected>Priority (High to Low)</option>
        {:else}
        <option value="ranking-desc">Priority (High to Low)</option>
        {/if}
        {#if sort == 'asc'}
        <option value="asc" selected>Price (Low to High)</option>
        {:else}
        <option value="asc">Price (Low to High)</option>
        {/if}
        {#if sort == 'desc'}
        <option value="desc" selected>Price (High to Low)</option>
        {:else}
        <option value="desc">Price (High to Low)</option>
        {/if}
        {#if sort == 'checked'}
        <option value="checked" selected>Claimed only</option>
        {:else}
        <option value="checked">Claimed only</option>
        {/if}
        {#if sort == 'unchecked'}
        <option value="unchecked" selected>Unclaimed only</option>
        {:else}
        <option value="unchecked">Unclaimed only</option>
        {/if}
    </select>
    <Button type="submit" href="">Sort</Button>
</form>
<br>
<ul class="mx-1 grid sm:grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5">
    {#each items as item}
        {#if item.checked}
            <li class="m-4 bg-transparent"> 
                <div class="bg-white dark:bg-extradarkgrey border border-darkgrey pb-5 px-5 rounded-md w-full min-w-full break-words">
                    <img src={item.img} alt={item.name} class="object-scale-down rounded-b-md rounded-t-none w-full rounded-md p-0 m-0" loading="lazy" />
                    <s class="dark:decoration-white decoration-black">
                    <h2 class="text-xl text-theme pt-2">{item.name} <a class="text-darkgrey hover:underline" href={`/item/${item._id}`}>#</a></h2>
                    <p class="text-black dark:text-white">{item.description}</p>
                    </s>
                    <i class="text-darkgrey dark:text-grey">${item.price}</i>
                    {#if item.size !== ""}
                    <p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-white">Size: {item.size}</p>
                    {:else}
                    <p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-white">Size: N/A</p>
                    {/if}
                    <p class="text-black dark:text-white">Priority: {item.ranking}/10</p>
                    <div class="my-2">
                        {#if item.link.length > 0}
                            <p class="inline dark:text-grey text-black">Buy from:</p>
                            <Button type="link" href={item.link[0].url}>{item.link[0].name}</Button> {#if item.link.length > 1}<i class="inline text-darkgrey dark:text-grey dark:hover:text-white">+{item.link.length - 1} more links...</i> <a class="text-black dark:text-white" href="/item/{item._id}">(see all)</a>{/if}
                        {/if}
                    </div>
                    <form class="mb-0" id={item._id} on:submit|preventDefault={updateItem}>
                        <input type="hidden" name="id" value={item._id} />
                        <label for="checkbox" class="text-black dark:text-white">Claimed:</label>
                        <input name={`checkbox-${item._id}`} id={`checkbox-${item._id}`} type="checkbox" checked={item.checked} class="accent-theme mr-1" />
                        <Button type="submit" href="">Update</Button>
                        <i class="text-darkgrey" id={`msg-${item._id}`}></i>
                    </form>
                </div>
            </li>
        {:else}
        <li class="m-2 bg-transparent"> 
            <div class="bg-grey dark:bg-reallydarkgrey border border-darkgrey pb-5 px-5 rounded-md w-full min-w-full break-words">
                <img src={item.img} alt={item.name} class="object-scale-down rounded-b-md rounded-t-none w-full rounded-md p-0 m-0" />
                <h2 class="text-xl text-theme pt-2">{item.name} <a class="text-darkgrey hover:underline" href={`/item/${item._id}`}>#</a></h2>
                <p class="text-black dark:text-white">{item.description}</p>
                <i class="text-darkgrey dark:text-grey">${item.price}</i>
                {#if item.size !== ""}
                <p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-white">Size: {item.size}</p>
                {:else}
                <p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-white">Size: N/A</p>
                {/if}
                <p class="text-black dark:text-white">Priority: {item.ranking}/10</p>
                <div class="my-2">
                    {#if item.link.length > 0}
                        <p class="inline dark:text-grey text-black">Buy from:</p>
                        <Button type="link" href={item.link[0].url}>{item.link[0].name}</Button> {#if item.link.length > 1}<i class="inline text-darkgrey dark:text-grey dark:hover:text-white">+{item.link.length - 1} more links...</i> <a class="text-black dark:text-white" href="/item/{item._id}" target="_blank" rel="noopener noreferrer">(see all)</a>{/if}
                    {/if}
                </div>
                <form class="mb-0" id={item._id} on:submit|preventDefault={updateItem}>
                    <input type="hidden" name="id" value={item._id} />
                    <label for="checkbox" class="text-black dark:text-white">Claimed:</label>
                    <input name={`checkbox-${item._id}`} id={`checkbox-${item._id}`} type="checkbox" checked={item.checked} class="accent-theme mr-1" />
                    <Button type="submit" href="">Update</Button>
                    <i class="text-darkgrey" id={`msg-${item._id}`}></i>
                </form>
            </div>
        </li>
        {/if}
    {/each}
</ul>
<MetaTags
  title={`Simeon's list`}
  description={`Simeon's wishlist`}
  openGraph={{
    url: $page.url,
    title: `Simeon's list`,
    description: `Simeon's wishlist`,
    site_name: 'Simeon\'s list'
  }}
  twitter={{
    cardType: 'summary_large_image',
    title: `Simeon's list`,
    description: 'Simeon\'s wishlist',
  }}
/>
