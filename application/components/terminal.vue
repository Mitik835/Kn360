<template>
  <div id="terminal">
    <div :class="{collapsed: !open}"></div>
  </div>
</template>

<script>
import { Terminal } from 'xterm'

export default {
  name: 'terminal',
  emits: ['new-message'],
  props: ['open', 'socket'],
  data() {
    return {
      _terminal: null
    }
  },
  mounted() {
    this.terminal = new Terminal({ disableStdin: true, rows: 12, cols: 104 })
    this.terminal.open(document.querySelector('#terminal > div'))
  }
}
</script>

<style>
#terminal {
	position: absolute;
	bottom: 0px;
	width: 100%;
	opacity: 0.85;
}
#terminal:hover {
	opacity: 0.98;
}

#terminal > .collapsed {
	overflow: hidden;
	height: 30px;
}
</style>