<script>
    import { MetaTags } from 'svelte-meta-tags';
    import { page } from '$app/stores';
    export let item
    export let loggedIn
    import Button from '../../../lib/components/Button.svelte'
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
    async function deleteItem(e) {
        const id = await e.target.id
        const msg = document.getElementById(`del-${id}`)
        msg.innerText = 'deleting...'
        const res = await fetch(`/item/${id}/`, {
            method: 'DELETE',
            headers: {
                'Authorization': document.cookie,
                'Accept': 'application/json'
            }
        })
        const json = await res.json()
        msg.innerText = `deleted`
        setTimeout(() => {
            msg.innerText = ""
            window.location.href = '/list'
        }, 300)
    }
</script>
<h1 class="text-3xl text-theme">{item.name}</h1>
<img src={item.img} alt={item.name} class="sm:max-w-screen lg:max-w-lg mx-2 p-3" />
<p class="text-lg pt-2 text-black dark:text-grey">{item.description}</p>
<i class="text-darkgrey">${item.price}</i>
{#if item.size !== ""}
<p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-grey">Size: {item.size}</p>
{:else}
<p class="underline decoration-solid decoration-theme decoration-2 text-black dark:text-grey">Size: N/A</p>
{/if}
<div class="mt-4 grid place-items-center">
    <b class="text-black dark:text-grey">Links:</b>
</div>
<div class="mt-2 mb-4 mx-4 grid place-items-center">
{#if item.link.length > 0}
{#each item.link as link}
    <div class="m-2">
        <Button type="link" href={link.url}>{link.name}</Button>
    </div>
{/each}
{:else}
<i class="text-black dark:text-grey">no links :(</i>
{/if}
</div>
<form class="mb-0" id={item._id} on:submit|preventDefault={updateItem}>
    <input type="hidden" name="id" value={item._id} />
    <label for="checkbox" class="text-black dark:text-grey">Claimed:</label>
    <input name={`checkbox-${item._id}`} id={`checkbox-${item._id}`} type="checkbox" checked={item.checked} class="accent-theme mr-1" />
    <Button type="submit" href="">Update</Button>
    <i class="text-darkgrey" id={`msg-${item._id}`}></i>
</form>
{#if loggedIn}
    <form on:submit|preventDefault={deleteItem} id={item._id}>
        <div class="mt-2">
            <input type="hidden" value={item._id} />
            <Button href="" type="submit">Delete Item</Button> <i class="text-darkgrey dark:text-grey" id={`del-${item._id}`}></i>
        </div>
    </form>
{/if}
<MetaTags
  title={`Simeon's list - ${item.name}`}
  description={`Simeon's wishlist - ${item.name}`}
  openGraph={{
    url: $page.url,
    title: `Simeon's list - ${item.name}`,
    description: `Simeon's wishlist - ${item.name}`,
    images: [
      {
        url: item.img,
        alt: item.name
      }
    ],
    site_name: 'Simeon\'s list'
  }}
  twitter={{
    cardType: 'summary_large_image',
    title: `Simeon's list - ${item.name}`,
    description: 'Simeon\'s wishlist',
    image: item.img,
    imageAlt: item.name
  }}
/>
