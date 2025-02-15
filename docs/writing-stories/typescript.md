---
title: 'Writing stories in TypeScript'
---

Writing your stories in [TypeScript](https://www.typescriptlang.org/) makes you more productive. You don't have to jump between files to look up component props. Your code editor will alert you about missing required props and even autocomplete prop values, just like when using your components within your app. Plus, Storybook infers those component types to auto-generate the [Controls](../api/doc-block-controls.md) table.

Storybook has built-in TypeScript support, so you can get started with zero configuration required.

## Typing stories with `Meta` and `StoryObj`

When writing stories, there are two aspects that are helpful to type. The first is the [component meta](./introduction.md#default-export), which describes and configures the component and its stories. In a [CSF file](../api/csf.md), this is the default export. The second is the [stories themselves](./introduction.md#defining-stories).

Storybook provides utility types for each of these, named `Meta` and `StoryObj`. Here's an example CSF file using those types:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'angular/typed-csf-file.ts.mdx',
    'web-components/typed-csf-file.ts.mdx',
    'common/typed-csf-file.ts.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

### Props type parameter

`Meta` and `StoryObj` types are both [generics](https://www.typescriptlang.org/docs/handbook/2/generics.html#working-with-generic-type-variables), so you can provide them with an optional prop type parameter for the component type or the component's props type (e.g., the `typeof Button` portion of `Meta<typeof Button>`). By doing so, TypeScript will prevent you from defining an invalid arg, and all [decorators](./decorators.md), [play functions](./play-function.md), or [loaders](./loaders.md) will type their function arguments.

The example above passes a component type. See [**Typing custom args**](#typing-custom-args) for an example of passing a props type.

## Using `satisfies` for better type safety

<IfRenderer renderer={['angular', 'web-components']}>

<Callout variant="info">

We are not yet able to provide additional type safety using the `satisfies` operator with Angular and Web components.

<details>
<summary>More info</summary>

Both Angular and Web components utilize a class plus decorator approach. The decorators provide runtime metadata, but do not offer metadata at compile time.

As a result, it appears impossible to determine if a property in the class is a required property or an optional property (but non-nullable due to a default value) or a non-nullable internal state variable.

For more information, please refer to [this discussion](https://github.com/storybookjs/storybook/discussions/20988).

</details>

</Callout>

</IfRenderer>

If you are using TypeScript 4.9+, you can take advantage of the new [`satisfies`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html) operator to get stricter type checking. Now you will receive type errors for missing required args, not just invalid ones.

Using `satisfies` to apply a story's type helps maintain type safety when sharing a `play` function across stories. Without it, TypeScript will throw an error that the `play` function may be undefined. The `satisfies` operator enables TypeScript to infer whether the play function is defined or not.

Finally, use of `satisfies` allows you to pass `typeof meta` to the `StoryObj` generic. This informs TypeScript of the connection between the `meta` and `StoryObj` types, which allows it to infer the `args` type from the `meta` type. In other words, TypeScript will understand that args can be defined both at the story and meta level and won't throw an error when a required arg is defined at the meta level, but not at the story level.

## Typing custom args

Sometimes stories need to define args that aren’t included in the component's props. For this case, you can use an [intersection type](https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types) to combine a component's props type and your custom args' type. For example, here's how you could use a `footer` arg to populate a child component:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'react/page-story-slots.ts.mdx',
    'vue/page-story-slots.2.ts.mdx',
    'vue/page-story-slots.3.ts.mdx',
    'angular/page-story-slots.ts.mdx',
    'web-components/page-story-slots.ts.mdx',
    'solid/page-story-slots.ts.mdx',
  ]}
  usesCsf3
  csf2Path="writing-stories/args#snippet-page-story-slots"
/>

<!-- prettier-ignore-end -->

<IfRenderer renderer={['vue']}>

### Vue specific tips

Vue has excellent support for TypeScript, and we have done our utmost to take advantage of that in the stories files. For example, consider the following strongly typed Vue3 single file component (SFC):

```html
<script setup lang="ts">
  defineProps<{ count: number; disabled: boolean }>();

  const emit = defineEmits<{
    (e: 'increaseBy', amount: number): void;
    (e: 'decreaseBy', amount: number): void;
  }>();
</script>

<template>
  <div class="card">
    {{ count }}
    <button @click="emit('increaseBy', 1)" :disabled="disabled">Increase by 1</button>
    <button @click="$emit('decreaseBy', 1)" :disabled="disabled">Decrease by 1</button>
  </div>
</template>
```

You can type check SFC files with vue-tsc and get editor support in VSCode by installing the [Vue Language Features (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) and [TypeScript Vue Plugin](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin) extensions.

This setup will add type support for `*.vue` imports to your `*.stories.ts` files, providing the same type safety and autocomplete features.

</IfRenderer>

<IfRenderer renderer={['svelte']}>

### Svelte specific tips

Svelte offers excellent TypeScript support for .svelte files. For example, consider the following component. You can run type checks using svelte-check and add VSCode editor support with the [Svelte for VSCode extension](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode&ssr=false#overview).

```html
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let count: number;
  export let disabled: boolean;

  const dispatch = createEventDispatcher();
</script>

<div class="card">
  {count}
  <button on:click={() => dispatch('increaseBy', 1)} {disabled}> Increase by 1 </button>
  <button on:click={() => dispatch('decreaseBy', 1)} {disabled}> Decrease by 1 </button>
</div>
```

The same setup works with Svelte stories files too, providing both type safety and autocompletion.

</IfRenderer>
