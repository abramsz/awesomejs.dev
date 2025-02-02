<template>
  <component
    :is="component"
    v-bind="$attrs"
    :type="type"
    :tabindex="ghost ? -1 : 0"
    role="button"
    :aria-disabled="ghost"
    class="inline-block cursor-pointer relative rounded select-none outline-none"
    :class="{
      'pointer-events-none opacity-50': ghost,
      'text-center': align === 'center',
    }"
    @click.capture="handleClick"
    @click.capture.native="handleClick"
  >
    <div
      class="flex items-center rounded"
      :class="{
        'opacity-0': loading,
        'justify-center': align === 'center',
      }"
    >
      <i
        v-if="iconLeft"
        class="material-icons text-lg mr-2"
      >{{ iconLeft }}</i>
      <slot />
      <i
        v-if="iconRight"
        class="material-icons text-lg ml-2"
      >{{ iconRight }}</i>
    </div>

    <SubmitAnimation
      v-if="loading"
      class="absolute inset-0"
    />
  </component>
</template>

<script>
import SubmitAnimation from './SubmitAnimation.vue'

export default {
  components: {
    SubmitAnimation,
  },

  inheritAttrs: false,

  props: {
    iconLeft: {
      type: String,
      default: null,
    },

    iconRight: {
      type: String,
      default: null,
    },

    type: {
      type: String,
      default: 'button',
    },

    loading: {
      type: Boolean,
      default: false,
    },

    disabled: {
      type: Boolean,
      default: false,
    },

    align: {
      type: String,
      default: 'center',
    },
  },

  computed: {
    component () {
      if (this.$attrs.to) {
        return 'router-link'
      } else if (this.$attrs.href) {
        return 'a'
      } else {
        return 'button'
      }
    },

    ghost () {
      return this.disabled || this.loading
    },
  },

  methods: {
    handleClick (event) {
      if (this.ghost) {
        event.preventDefault()
        event.stopPropagation()
        event.stopImmediatePropagation()
      } else {
        this.$emit('click', event)
      }
    },
  },
}
</script>
