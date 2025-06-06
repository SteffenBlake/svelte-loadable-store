# Fork Acknowledgement ⚠️
This project is a fork of the original work by [Nikita Galaiko](https://github.com/ngalaiko/svelte-loadable-store). I have updated and maintained it further to work with Svelte 5. Many thanks to Nikita for the foundation and effort put into the original library.

# Svelte Loadable Store

[![npm](https://img.shields.io/npm/v/svelte-loadable-store)](https://www.npmjs.com/package/svelte-loadable-store)
[![license](https://img.shields.io/github/license/SteffenBlake/svelte-loadable-store)](https://raw.githubusercontent.com/SteffenBlake/svelte-loadable-store/master/LICENSE)
[![test](https://github.com/SteffenBlake/svelte-loadable-store/actions/workflows/test.yaml/badge.svg)](https://github.com/SteffenBlake/svelte-loadable-store/actions/workflows/test.yaml)

This library provides wrappers for vanilla svelte stores to simplify consumption of async APIs.

## Types

Store value can be in three states: `loading`, `loaded` or `error`. Core types reflect that:

```typescript
type Loadable<T> = Loading | Loaded<T>;
type Loading = { isLoading: true };
type Loaded<T> = { isLoading: false; value: T } | { isLoading: false; error: any };
```

## writable / readable

`writable` and `readable` allow to define stores that are initialized asyncronously, for example:

```typescript
import { readable } from 'svelte-loadable-store';

const start = performance.now();

const delay = (timeout: number) =>
	new Promise<number>((resolve) => setTimeout(() => resolve(performance.now() - start), timeout));

const store = readable(delay(100), (set) => {
	delay(200).then((value) => set(value));
});

store.subscribe(console.log);

/*
 * prints:
 * { isLoading: true }
 * { isLoading: false, value: 101 }
 * { isLoading: false, value: 201 }
 */
```

`writable` is exactly the same, just allows to `.set` and `.update` according to the Svelte's contract.

## derived

The real power comes with `derived` stores. Derived function will be executed only after all of the
input stores are loaded successfully. You can also derive asyncronously.

```typescript
import { writable, derived } from 'svelte-loadable-store';

const start = performance.now();

const delay = (timeout: number) =>
	new Promise<number>((resolve) => setTimeout(() => resolve(performance.now() - start), timeout));

const first = writable(delay(100));
const second = writable(delay(200));
const third = derived([first, second], async ([first, second]) => first + second);

first.subscribe((v) => console.log('first', v));
second.subscribe((v) => console.log('second', v));
third.subscribe((v) => console.log('third', v));

/*
 * prints:
 * first { isLoading: true }
 * second { isLoading: true }
 * third { isLoading: true }
 * first { isLoading: false, value: 101 }
 * second { isLoading: false, value: 201 }
 * third { isLoading: false, value: 302 }
 */
```

## error handling

Library exposes a few handy functions to use when working with store values, for example:

```svelte
<script lang="ts">
    import { writable, derived, Loaded } from 'svelte-loadable-store'

    const fetchData = (): Promise<Data> => fetch('https://api.example.com/data').then(response => response.json())

    const data = writable(fetchData)
</script>

{#if $data.isLoading}
    loading..
{:else Loaded.isError($data)}
    error: {$data.error}
{:else}
    data: {$data.value}
{/if}
```

## promisify

A util funciton that converts a store into promise that will resolve with the first loaded value.

```ts
import { readable, promisify } from 'svelte-loadable-store';

const delay = (timeout: number) =>
	new Promise<number>((resolve) => setTimeout(() => resolve(performance.now() - start), timeout));

const store = readable(delay(100));

promisify(store).then(console.log);

/*
 * prints:
 * 101
 */
```

## Acknowledgement

This library is inspired by [@square/svelte-store](https://github.com/square/svelte-store). I have been using it myself
before writing this one, but found it having quite a complex interface and faced [some issues](https://github.com/square/svelte-store/issues/61).

Inspiration for type definitions comes from this [nanostores issue](https://github.com/orgs/nanostores/discussions/150).

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
