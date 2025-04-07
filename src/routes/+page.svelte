<script>
// @ts-nocheck

	import Gun from "gun/gun"
	var gun = Gun({peers:['https://gundbpeer-63a4d85a63b9.herokuapp.com/gun']}).get('todos');
	
	// Create a local store to cache data from GUN
	let store = {}
	
	// Creates a listener that iterates over keys found in the "todo" node in GUN
	gun.map().on(function(todo, key) {
		if (todo) {
			// Updates the store with the new value
			store[key] = todo
		} else {
			// A key may contain a null value (if data has been deleted/set to null)
			// if so, we remove the item from the store
			delete store[key]
			store = store
		}
	})
	// The below lines listens for updates in the store and creates
	// more convenient variables for use in markup
	$: todos = Object.entries(store)
	$: done = todos.filter(([key, todo]) => todo.done).length
	
	// Actions that write data to GUN
	const create = key => gun.set(key).put({ title: key, done: false })
	const update = (key, value) => gun.get(key).get("done").put(value)
	const remove = key => gun.get(key).put(null)
</script>

<input placeholder="add todo" on:change={e => create(e.target.value) && (e.target.value = "")} />
completed {done}/{todos.length}

{#each todos as [key, todo]}
<div id={key}>
	<input type="checkbox" checked={todo.done} on:change={() => update(key, !todo.done)} />
	{todo.title} <a href="/" on:click|preventDefault={() => remove(key)}>remove</a> {todo.done ? "=" : "="}
</div>
{/each}
