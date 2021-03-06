# Using Vue in Markdown

## Browser API Access Restrictions

Because VuePress applications are server-rendered in Node.js when generating static builds, any Vue usage must conform to the [universal code requirements](https://ssr.vuejs.org/en/universal.html). In short, make sure to only access Browser / DOM APIs in `beforeMounted` or `mounted` hooks.

If you are using or demoing a component that is not SSR friendly (for example containing custom directives), you can wrap them inside the built-in `<ClientOnly>` component:

``` md
<ClientOnly>
  <NonSSRFriendlyComponent/>
</ClientOnly>
```

Note this does not fix components or libraries that access Browser APIs **on import** - in order to use code that assumes a browser environment on import, you need to dynamically import them in proper lifecycle hooks:

``` vue
<script>
export default {
  mounted () {
    import('./lib-that-access-window-on-import').then(module => {
      // use code
    })
  }
}
</script>
```

## Templating

### Interpolation

Each markdown file is first compiled into HTML and then passed on as a Vue component to `vue-loader`. This means you can use Vue-style interpolation in text:

**Input**

``` md
{{ 1 + 1 }}
```

**Output**

<pre><code>{{ 1 + 1 }}</code></pre>

### Directives

Directives also work:

**Input**

``` md
<span v-for="i in 3">{{ i }} </span>
```

**Output**

<pre><code><span v-for="i in 3">{{ i }} </span></code></pre>

### Access to Site & Page Data

The compiled component does not have any private data but do have access to the [site metadata](./custom-themes.md#site-and-page-metadata). For example:

**Input**

``` md
{{ $page }}
```

**Output**

``` json
{
  "path": "/using-vue.html",
  "title": "Using Vue in Markdown",
  "frontmatter": {}
}
```

## Escaping

By default, fenced code blocks are automatically wrapped with `v-pre`. If you want to display raw mustaches or Vue-specific syntax inside inline code snippets or plain text, you need to wrap a paragraph with the `v-pre` custom container:

**Input**

``` md
::: v-pre
`{{ This will be displayed as-is }}`
:::
```

**Output**

::: v-pre
`{{ This will be displayed as-is }}`
:::

## Using Components

Any `*.vue` file found in `.vuepress/components` are automatically registered as global async components. For example:

```
.
└─ .vuepress
   └─ components
      ├─ demo-1.vue
      └─ OtherComponent.vue
```

Inside any markdown file you can then directly use the components (names are inferred from filenames):

``` md
<demo-1/>
<OtherComponent/>
```

<demo-1></demo-1>

<OtherComponent/>

::: warning IMPORTANT
Make sure a custom component's names either contains a hyphen or is in PascalCase. Otherwise it will be treated as an inline element and wrapped inside a `<p>` tag, which will lead to hydration mismatch because `<p>` does not allow block elements to be placed inside it.
:::

## Script & Style Hoisting

Sometimes you may need to apply some JavaScript or CSS only to the current page. In those cases you can directly write root-level `<script>` or `<style>` blocks in the markdown file, and they will be hoisted out of the compiled HTML and used as the `<script>` and `<style>` blocks for the resulting Vue single-file component.

<p class="demo" :class="$style.example"></p>

<style module>
.example {
  color: #41b883;
}
</style>

<script>
export default {
  mounted () {
    document.querySelector(`.${this.$style.example}`)
      .textContent = 'This is rendered by inline script and styled by inline CSS'
  }
}
</script>
