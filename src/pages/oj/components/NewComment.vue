<template>
  <Card class="comment-card">
    <p slot="title">New Comment</p>
    <div slot="extra">
      <Button @click="submit" type="text">Submit</Button>
      <Button @click="handleCancel" type="text">Cancel</Button>
    </div>
    <Input
      type="textarea"
      :autosize="{ minRows: 2, maxRows: 4}"
      v-model="message">
    </Input>
  </Card>
</template>

<script>
  import api from '@oj/api'

  export default {
    name: 'NewComment',
    props: {
      doc: Object,
      line: Number
    },
    data () {
      return {
        widget: {},
        message: ''
      }
    },
    methods: {
      submit () {
        api.createComment(this.$route.params.id, this.line, this.message).then(res => {
          this.$emit('created', res.data.data)
        })
      },
      handleCancel () {
        this.$emit('canceled')
      }
    },
    computed: {
      options () {
        return {
          coverGutter: true,
          noHScroll: true
        }
      }
    },
    mounted () {
      this.widget = this.doc.addLineWidget(this.line, this.$el, this.options)
    },
    beforeDestroy () {
      this.widget.clear()
    },
    watch: {
      'line' () {
        this.widget.clear()
        this.widget = this.doc.addLineWidget(this.line, this.$el, this.options)
      }
    }
  }
</script>

<style lang="less" scoped>
  .comment-card {
    margin: 10px;
  }
</style>
