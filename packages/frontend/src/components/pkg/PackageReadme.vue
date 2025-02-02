<template>
  <LoadingIndicator
    v-if="$apollo.loading"
    class="p-8"
  />
  <div
    v-else
    class="readme relative pb-8 mt-4 border-t-2 border-gray-800"
    :class="{
      expand: expanded,
      'overflow-y-hidden': !expanded
    }"
  >
    <div
      ref="render"
      class="markdown pt-8 pb-64"
      v-html="pkg.readme"
    />

    <div
      v-if="!$responsive.lg"
      class="action-overlay absolute bottom-0 left-0 w-full flex justify-center pt-6 pb-2"
      @click="expand = !expand"
    >
      <BaseButton
        class="p-1 bg-gray-800"
      >
        <i class="material-icons">{{ expand ? 'expand_less' : 'expand_more' }}</i>
      </BaseButton>
    </div>
  </div>
</template>

<script>
import LoadingIndicator from '../LoadingIndicator.vue'
import gql from 'graphql-tag'

const GOOGLE_IMG_PROXY = 'https://images1-focus-opensocial.googleusercontent.com/gadgets/proxy?container=focus&refresh=2592000&url='

export default {
  components: {
    LoadingIndicator,
  },

  props: {
    packageId: {
      type: String,
      required: true,
    },
  },

  data () {
    return {
      expand: false,
    }
  },

  apollo: {
    pkg: {
      query: gql`
        query PackageReadme ($id: ID!) {
          pkg: package (id: $id) {
            id
            readme
          }
        }
      `,
      variables () {
        return {
          id: this.packageId,
        }
      },
    },
  },

  computed: {
    expanded () {
      return this.expand || this.$responsive.lg
    },
  },

  watch: {
    'pkg.readme': 'processRender',
  },

  methods: {
    async processRender () {
      await this.$nextTick()
      const render = this.$refs.render

      // Images
      const imgs = render.querySelectorAll('img')
      for (const el of imgs) {
        const img = new Image()
        img.crossOrigin = 'Anonymous'
        img.onload = () => this.onImgLoad(img, el)
        img.src = `${GOOGLE_IMG_PROXY}${el.src}`
      }
    },

    onImgLoad (img, el) {
      // Empty
      if (img.width <= 1 || img.height <= 1) return

      // Badges
      if (img.height === 20) return

      // Calculate brightness
      const canvas = document.createElement('canvas')
      canvas.width = img.width
      canvas.height = img.height

      const ctx = canvas.getContext('2d')
      ctx.drawImage(img, 0, 0)

      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      const data = imageData.data
      let r, g, b, avg
      let colorSum = 0

      for (let x = 0, len = data.length; x < len; x += 4) {
        r = data[x]
        g = data[x + 1]
        b = data[x + 2]

        avg = Math.floor((r + g + b) / 3)
        colorSum += avg
      }

      const brightness = Math.floor(colorSum / (img.width * img.height))
      if (brightness < 60) {
        el.classList.add('ally-bg')

        if (img.width === img.height && img.width <= 64) {
          el.classList.add('avatar')
        }
      }
    },
  },
}
</script>

<style lang="postcss" scoped>
.readme {
  &:not(.expand) {
    max-height: calc(100vh - 500px);
  }
}

.action-overlay {
  background: linear-gradient(to bottom, transparent, theme('colors.gray.900'));
}
</style>
