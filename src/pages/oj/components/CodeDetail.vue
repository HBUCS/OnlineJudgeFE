<template>
  <div>
    <codemirror
      @update="getComments"
      :options="options"
      ref="myEditor"
      v-model="code"
      @gutterClick="gutterClick">
    </codemirror>

    <comment
      v-for="(comment, index) in comments"
      :key="comment.id"
      :comment="comment"
      :index="index"
      :doc="doc"
      @changed="changed"
      @deleted="deleted">
    </comment>

    <new-comment
      v-if="isCreate"
      :doc="doc"
      :line="newCommentLine"
      @created="created"
      @canceled="isCreate = false">
    </new-comment>
  </div>
</template>

<script>
  import api from '@oj/api'
  import utils from '@/utils/utils'
  import { codemirror } from 'vue-codemirror'
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/mode/clike/clike.js'
  import 'codemirror/mode/python/python.js'

  import Comment from './Comment'
  import NewComment from './NewComment'

  export default {
    name: 'CodeDetail',
    components: {
      codemirror,
      Comment,
      NewComment
    },
    props: {
      code: String,
      language: {
        type: String,
        default: 'C++'
      }
    },
    data () {
      return {
        comments: [],
        mode: {
          'C++': 'text/x-csrc'
        },
        isCreate: false,
        newCommentLine: 0
      }
    },
    computed: {
      editor () {
        return this.$refs.myEditor.codemirror
      },
      doc () {
        return this.$refs.myEditor.codemirror.getDoc()
      },
      options () {
        return {
          mode: 'text/x-csrc',
          lineNumbers: true,
          lineWrapping: true,
          readOnly: true
        }
      }
    },
    methods: {
      gutterClick (instance, line, gutter, clickEvent) {
        this.newCommentLine = line
        this.isCreate = true
      },
      created (comment) {
        this.comments.push(comment)
        this.isCreate = false
      },
      changed (index, comment) {
        this.comments[index] = comment
      },
      deleted (index) {
        this.comments.splice(index)
      },
      getComments () {
        api.getComment(this.$route.params.id).then(res => {
          this.comments = res.data.data
        })
      }
    },
    mounted () {
      utils.getLanguages().then(languages => {
        let mode = {}
        languages.forEach(lang => {
          mode[lang.name] = lang.content_type
        })
        this.mode = mode
        this.editor.setOption('mode', this.mode[this.language])
      })
    }
  }
</script>

<style lang="less">
  .CodeMirror {
    border: 1px solid #eee;
    height: auto;
  }
</style>
