<template>
  <div class="uk-form-row">
    <input v-model="model.classes" class="uk-width-1-1" >
  </div>
</template>

<script>
import { createGenerator } from '@unocss/core'
import presetWind from '@unocss/preset-wind'
import debounce from 'lodash-es/debounce'

export default {
  name: 'TailwindBlockClassesPlugin',
  mixins: [window.Storyblok.plugin],
  data () {
    return {
      uno: null,
    }
  },
  methods: {
    initWith () {
      return {
        plugin: 'tailwind-block-classes',
        classes: '',
        css: '',
      }
    },
    pluginCreated () {
      if (this.options && this.options.themeUrl) {
        fetch(this.options.themeUrl)
          .then(res => res.json())
          .then(theme => {
            if (theme.screens) {
              theme.breakpoints = theme.screens
              delete theme.screens
            }
            this.uno = createGenerator({ presets: [presetWind()], theme })
            this.generateStyles()
          })
      } else {
        this.uno = createGenerator({ presets: [presetWind()] })
        this.generateStyles()
      }
    },
    generateStyles: debounce(async function () {
      let css = ''
      if (this.uno && this.model.classes) {
        const selector = `sb-${this.blockId}_SELECTOR_`
        this.uno.setConfig({
          ...this.uno.config,
          shortcuts: {
            [selector]: this.model.classes
          }
        })
        const result = await this.uno.generate(selector, { scope: `.story-${this.storyId}` })
        css = result.css
      }
      this.model.css = css
    }, 500),
  },
  watch: {
    model: {
      handler (value) {
        this.$emit('changed-model', value)
      },
      deep: true,
    },
    'model.classes': function () {
      this.generateStyles()
    }
  }
}
</script>
