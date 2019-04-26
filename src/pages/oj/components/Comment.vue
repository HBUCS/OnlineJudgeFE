<template>
  <Card class="comment-card">
    <div slot="title">
      {{comment.username}} <span style="color: #c3cbd6;">{{comment.create_time | localtime}}</span>
    </div>
    <div slot="extra">
      <Button @click="submit" type="text" v-if="isEdit">Submit</Button>
      <Button @click="isEdit = true; message = comment.message" type="text" v-else>Edit</Button>
      <Button @click="isEdit = false" type="text" v-if="isEdit">Cancel</Button>
      <Button @click="del" type="text" v-else>Delete</Button>
    </div>
    <Input
      v-if="isEdit"
      type="textarea"
      :autosize="{ minRows: 2, maxRows: 4}"
      v-model="message">
    </Input>
    <div class="comment-content" v-else>
      <Avatar icon="ios-person" size="large"/>
      <p>{{comment.message}}</p>
    </div>
  </Card>
</template>

<script>
  import api from '@oj/api'

  export default {
    name: 'Comment',
    props: {
      doc: Object,
      comment: Object,
      index: Number
    },
    data () {
      return {
        widget: {},
        isEdit: false,
        message: ''
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
    methods: {
      submit () {
        api.editComment(this.comment.id, this.message).then(res => {
          this.$emit('changed', this.index, res.data.data)
          this.isEdit = false
        })
      },
      del () {
        api.deleteComment(this.comment.id).then(res => {
          this.$emit('deleted', this.index)
        })
      }
    },
    mounted () {
      this.widget = this.doc.addLineWidget(this.comment.line, this.$el, this.options)
    },
    beforeDestroy () {
      this.widget.clear()
    },
    updated () {
      this.widget.changed()
    }
  }
</script>

<style lang="less" scoped>
  .comment-card {
    margin: 10px;

    .comment-content {
      display: flex;
      width: 100%;

      p {
        padding-left: 16px;
        flex-grow: 1;
      }
    }
  }
</style>
