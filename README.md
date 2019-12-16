# vue-wrapped-component

> A functional Vue component that allows any component or element to be conditionally wrapped using another component or element.

## Installation

```bash
npm -i vue-wrapped-component
```

```bash
yarn add vue-wrapped-component
```

## Requirements

- `vue 2.6+` _(may work with older versions, just not tested)_
- JavaScript ES6 (or some compiler/loader that can handle ES6)

## Basic Usage

To install globally:
```js
import Vue from 'vue'
import WrappedComponent from "vue-wrapped-component"

Vue.use(WrappedComponent)
```

Otherwise, you can still use it at just the local component level:

**MyComponent.vue**
```vue
<template>
  <wrapped-component :wrap="aConditional">
    <some-component #wrapper />
    Components to be rendered either wrapped or unwrapped 
  </wrapped-component>
</template>

<script>
  import WrappedComponent from "vue-wrapped-component"
  
  export default {
    name: 'my-component',
    components: {
      WrappedComponent,
    }
  }
</script>
```

Use the special `#wrapper` slot syntax (`v-slot="wrapper"`) to denote which component should be the wrapper component.
This slot is required and this component will not work without it. All other components will be injected into this
wrapper component based on the the `wrap` conditional property.

## Props

| Name                  | Type          | Default       | Description                                  |
|-----------------------|---------------|---------------|----------------------------------------------|
| `tag`                 | `String`      | `'div'`       | The HTML tag to use when there is more than one root level wrapper or contents; see below. |
| `wrap`                | `Boolean`     | `false`       | The conditional for which to toggle the "wrapped" state and place the contents in the wrapper component. |
| `wrapRoot`            | `Boolean`     | `false`       | When there is more than one root level wrapper or contents, a wrapping element must be used. By default, if there's only one root level element, the wrapper component or the contents will be returned as is. Enabling this will always wrap the wrapper and contents, essentially turning `<wrapper-component>` into whatever `tag` is set as. |

## Use case

The primary purpose of this component is to assist in reducing the duplication of code in the rare use cases where
you need to render the exact same thing, but inside a wrapper which is based on a conditional.

Take for instance (using [bootstrap-vue](https://bootstrap-vue.js.org/docs/components/link)):

```vue
<b-link v-if="link" :to="link">{{ title }}</b-link>
<template v-else>{{ title }}</template>
```

Using `<wrapped-component>`, this can be written instead as:

```vue
<wrapped-component :wrap="link">
  <b-link :href="link" #wrapper />
  {{ title }}
</wrapped-component>
```

Now, you're probably thinking "Why? This looks like it's just adding unnecessary lines!"

Yes, if the content you're wishing to wrap is that basic, you would probably be better off just using the normal
`v-if` and `v-else` directives.

Where the power of this component really lies is when the contents (what you want to wrap, e.g. `{{ title }}`) is
far more complex than the above example.

What if your link needed an icon? Perhaps other additional components inside it as well?

A more complex example: a Shipping Address form (using [bootstrap-vue](https://bootstrap-vue.js.org/docs/components/form-group)) :

```vue
<b-form-group
  label-cols-lg="3"
  label="Shipping Address"
  label-size="lg"
  label-class="font-weight-bold pt-0"
  class="mb-0"
>
  <b-form-group
    label-cols-sm="3"
    label="Street:"
    label-align-sm="right"
    label-for="nested-street"
  >
    <b-form-input id="nested-street"></b-form-input>
  </b-form-group>

  <b-form-group
    label-cols-sm="3"
    label="City:"
    label-align-sm="right"
    label-for="nested-city"
  >
    <b-form-input id="nested-city"></b-form-input>
  </b-form-group>

  <b-form-group
    label-cols-sm="3"
    label="State:"
    label-align-sm="right"
    label-for="nested-state"
  >
    <b-form-input id="nested-state"></b-form-input>
  </b-form-group>

  <b-form-group
    label-cols-sm="3"
    label="Country:"
    label-align-sm="right"
    label-for="nested-country"
  >
    <b-form-input id="nested-country"></b-form-input>
  </b-form-group>

  <b-form-group
    label-cols-sm="3"
    label="Ship via:"
    label-align-sm="right" class="mb-0"
  >
    <b-form-radio-group
      class="pt-2"
      :options="['Air', 'Courier', 'Mail']"
    ></b-form-radio-group>
  </b-form-group>
</b-form-group>
```

#### Component `#wrapper` slot directive (`v-slot:wrapper`)

Now, imagine the entire form above needing to be rendered inside a modal for mobile purposes?

You would have to duplicate the entire form (and all the complexities that come with it) to place it inside the
modal component and throw some `v-if` and `v-else` directives like above.

This not only leads to code duplication, but also runs the risk of code fragmentation.

`<wrapped-component>` helps with this by intelligently injecting the contents (anything not flagged with the `#wrapper`
slot directive) into the first component you have specified using the `#wrapper` slot directive. The wrapper component
is then toggled based on the the value you provide the `wrap` property.

```vue
<wrapped-component :wrap="isMobile">
  <b-modal #wrapper />
  <b-form-group>
    <!-- All the form code from above will be rendered as is here if not wrapped and inside the modal if it is. -->  
  </b-form-group>
</wrapped-component>
```

That's great, but what if you need to also render a button that will open said modal (since the modal is hidden by
default)? You can simply add another `#wrapper` slot directive onto the component you wish to have rendered when
wrapped.

The first component flagged with the `#wrapper` slot directive will be used as the primary "wrapper" (the component
where all the non-wrapper components are injected into). Any components flagged with the `#wrapper` slot directive
after that are purely supplemental and will be rendered as sibling components along side, not inside of, the primary
wrapper component.

```vue
<wrapped-component :wrap="isMobile">

  <b-modal #wrapper ref="modal" />
  <b-button #wrapper @click="$refs.modal.show()">
    {{ $t('Edit Shipping Address') }}
  </b-button>

  <b-form-group>
    <!-- All the form code from above will be rendered as is here if not wrapped and inside the modal if it is. -->  
  </b-form-group>

</wrapped-component>
```

#### Nested named and scoped slots caveat

Currently, there is a caveat when using nested named and scoped slots. This caveat only really applies if you actually
need to override or implement a named or scoped slot that a component provides and, unfortunately, the `v-slot`
directive does not easily allow this directly with components. There are two ways to deal with this scenario:

1. You can use the deprecated `slot="name"` attribute instead (not really recommended, but will do in a pinch).
2. Place the components inside `<template #wrapper>` (recommended, see below).

```vue
<wrapped-component :wrap="isMobile">

  <b-modal #wrapper ref="modal">
    <template slot="modal-header">
      When using the #wrapper slot directive (v-slot:wrapper) on your component, any slots that component defines must
      use the deprecated slot="name" attribute to function properly. This is caveat of how nested named slots work in
      Vue. When this attribute is not longer used, you will have to resort to using the template method below. 
    </template>
  </b-modal>
  <b-button #wrapper @click="$refs.modal.show()">
    {{ $t('Edit Shipping Address') }}
  </b-button>

  <b-form-group>
    <!-- All the form code from above will be rendered as is here if not wrapped and inside the modal if it is. -->  
  </b-form-group>

</wrapped-component>
```

#### Template #wrapper (rendering multiple/complex components when wrapped)

This method, while not the prettiest, is likely the safest when you need explicit control over multiple/complex
components that also supply named and scoped slots. When using this method, all components should behave normally
and you can use whatever `v-slot` directives you need to on these components.

```vue
<wrapped-component :wrap="isMobile">

  <template #wrapper>
    <b-modal ref="modal">
      <template #modal-header>
        When using templates, your wrapper component's slots can be used like normal. 
      </template>
    </b-modal>
    <b-button @click="$refs.modal.show()">
      {{ $t('Edit Shipping Address') }}
    </b-button>
  </template>

  <b-form-group>
    <!-- All the form code from above will be rendered as is here if not wrapped and inside the modal if it is. -->  
  </b-form-group>

</wrapped-component>
```
