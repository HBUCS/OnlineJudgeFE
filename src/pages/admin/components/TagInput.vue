<template>
  <div>
    <el-tag
      :closable="true"
      :key="tag"
      @close="handleClose(tag)"
      v-for="tag in tags">
      {{tag}}
    </el-tag>
    <el-input
      @blur="handleInputConfirm"
      @keyup.enter.native="handleInputConfirm"
      class="input-new-tag"
      size="small"
      v-if="!fetchSuggestions && inputVisible"
      v-model="inputValue">
    </el-input>
    <el-autocomplete
      :fetch-suggestions="fetchSuggestions"
      :trigger-on-focus="false"
      @keyup.enter.native="handleInputConfirm"
      @select="handleInputConfirm"
      class="input-new-tag"
      size="small"
      v-if="fetchSuggestions && inputVisible"
      v-model="inputValue">
    </el-autocomplete>
    <el-button @click="showInput" class="button-new-tag" size="small" v-else>+ New One</el-button>
  </div>
</template>

<script>
  export default {
    name: 'TagInput',
    props: {
      tags: {
        type: Array,
        default: () => []
      },
      fetchSuggestions: {
        type: Function,
        default: null
      }
    },
    data () {
      return {
        inputVisible: false,
        inputValue: ''
      }
    },
    methods: {
      handleClose (tag) {
        this.$emit('delTag', tag)
      },
      showInput () {
        this.inputVisible = true
      },
      handleInputConfirm () {
        if (this.inputValue) {
          this.$emit('addTag', this.inputValue)
        }
        this.inputVisible = false
        this.inputValue = ''
      }
    }
  }
</script>

<style scoped>
  .el-tag + .el-tag {
    margin-left: 10px;
  }

  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }

  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>
