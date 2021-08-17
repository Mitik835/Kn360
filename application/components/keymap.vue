<script>
import KeyboardLayout from './keyboard-layout.vue'
import LayerSelector from './layer-selector.vue'
import Search from './search.vue'

import { loadIndexedKeycodes } from '../keycodes'
import {
  parseKeyBinding,
  updateKeyCode,
  encode
} from '../keymap'

export default {
  name: 'keymap',
  components: {
    'layer-selector': LayerSelector,
    'keyboard-layout': KeyboardLayout,
    'search': Search
  },
  props: ['layout', 'layers'],
  emits: ['keymap-updated'],
  provide() {
    return {
      onSelectKey: this.handleSelectKey
    }
  },
  data() {
    return {
      activeLayer: 0,
      indexedKeycodes: {},
      parsedKeymap: {},
      editing: null
    }
  },
  methods: {
    handleSelectKey(event) {
      this.editing = event
    },
    handleChangeBinding(code) {
      const { index, codeIndex, param } = this.editing
      const key = this.parsedKeymap[this.activeLayer][index]
      updateKeyCode(key, codeIndex, code, param, this.indexedKeycodes)
      this.editing = null
      this.$emit('keymap-updated', Object.assign({}, this.keymap, {
        layers: encode(this.parsedKeymap)
      }))
    }
  },
  async beforeMount() {
    const indexedKeycodes = await loadIndexedKeycodes()
    Object.assign(this.indexedKeycodes, indexedKeycodes)

    this.parsedKeymap = this.layers.map((layer, i) => {
      return layer.map((binding, j) => {
        const parsed = parseKeyBinding(binding, indexedKeycodes)
        return { layer: i, index: j, binding, parsed }
      })
    })
  }
}
</script>

<template>
  <div>
    <layer-selector
      :layers="layers"
      :activeLayer="activeLayer"
      @select="activeLayer = $event"
    />
    <keyboard-layout
      v-if="parsedKeymap[activeLayer]"
      :data-layer="activeLayer"
      :layout="layout"
      :keys="parsedKeymap[activeLayer]"
      class="active"
    />
    <search
      v-if="editing"
      :target="editing.target"
      :code="editing.code"
      :param="editing.param"
      @select="handleChangeBinding"
      @cancel="editing = null"
    />
  </div>
</template>