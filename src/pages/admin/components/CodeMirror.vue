<template>
  <codemirror
    :options="options"
    @blur="$emit('blur')"
    @input="onCodeChange"
    ref="myCm"
    v-model="currentValue"></codemirror>
</template>

<script>
  import { codemirror } from 'vue-codemirror'
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/mode/clike/clike.js'
  import 'codemirror/mode/python/python.js'
  import 'codemirror/mode/javascript/javascript.js'
  import 'codemirror/mode/lua/lua.js'
  import 'codemirror/theme/solarized.css'

  export default {
    name: 'CodeMirror',
    data () {
      return {
        currentValue: '',
        options: {
          mode: 'text/x-csrc',
          lineNumbers: true,
          lineWrapping: false,
          theme: 'solarized',
          tabSize: 4,
          line: true,
          foldGutter: true,
          gutters: ['CodeMirror-linenumbers', 'CodeMirror-foldgutter'],
          autofocus: false
        }
      }
    },
    components: {
      codemirror
    },
    props: {
      value: {
        type: String,
        default: ''
      },
      mode: {
        type: String,
        default: 'text/x-csrc'
      }
    },
    computed: {
      codemirror () {
        return this.$refs.myCm.codemirror
      }
    },
    mounted () {
      this.currentValue = this.value
      this.codemirror.setOption('mode', this.mode)
    },
    methods: {
      onCodeChange (newCode) {
        this.$emit('input', newCode)
      }
    },
    watch: {
      'value' (val) {
        if (this.currentValue !== val) {
          this.currentValue = val
        }
      },
      'mode' (newVal) {
        this.codemirror.setOption('mode', newVal)
      }
    }
  }
</script>

<style>
  .CodeMirror {
    height: auto !important;
  }

  .CodeMirror-scroll {
    min-height: 300px;
    max-height: 1000px;
  }
</style>
