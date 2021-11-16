<template>
  <div class="uk-form-row">
    <input v-model="model.classes" class="uk-width-1-1" >
  </div>
</template>

<script>
import { createGenerator } from '@unocss/core'
import presetUno from '@unocss/preset-uno/dist/index.js'
import debounce from 'lodash-es/debounce'

export default {
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
            this.uno = createGenerator({ presets: [presetUno()], theme })
            this.generateStyles()
          })
      } else {
        this.uno = createGenerator({ presets: [presetUno()] })
        this.generateStyles()
      }
    },
    generateStyles: debounce(async function () {
      let css = ''
      if (this.uno && this.model.classes) {
        const result = await this.uno.generate(new Set(this.model.classes.split(' ')))
        css = result.css
        if (css) {
          css = css.replace(/\/\*.*\*\/\n/gm, '')
          css = css.replace(/^\..*{/gm, '{')
          css = css.replace(/}\n{/gm, '')
          css = `.story-${this.storyId} .sb-${this.blockId}_SELECTOR_${css}`
        }
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
