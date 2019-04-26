<template>
  <el-form :model="form" :rules="rules" label-width="75px" ref="form">
    <el-form-item :label="$t('m.Tag')" prop="tags">
      <tag-input
        :fetchSuggestions="querySearch"
        :tags="form.tags"
        @addTag="addTag"
        @delTag="delTag">
      </tag-input>
    </el-form-item>
    <el-form-item :label="$t('m.Source')" prop="sources">
      <tag-input
        :tags="form.sources"
        @addTag="addSource"
        @delTag="delSource">
      </tag-input>
    </el-form-item>
    <el-form-item :label="$t('m.Difficulty')" prop="difficulty">
      <el-checkbox-group v-model="form.difficulty" size="small">
        <el-checkbox label="Low"></el-checkbox>
        <el-checkbox label="Mid"></el-checkbox>
        <el-checkbox label="High"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-form-item :label="$t('m.Number')" prop="number" required>
      <el-input-number v-model="form.number" size="small"></el-input-number>
    </el-form-item>
    <el-button :loading="loading" @click="submit" type="primary">Extract</el-button>
  </el-form>
</template>

<script>
  import TagInput from '../../components/TagInput'
  import api from '@admin/api'

  export default {
    name: 'extract-question',
    components: {
      TagInput
    },
    props: ['contestID'],
    data () {
      return {
        form: {
          sources: [],
          tags: [],
          difficulty: ['Low'],
          number: 1
        },
        rules: {
          number: [
            {required: true, message: 'Number is required', trigger: 'blur'},
            {type: 'number', min: 1, message: 'Quantity must be greater than zero', trigger: 'blur'}
          ]
        },
        loading: false
      }
    },
    methods: {
      addTag (value) {
        this.form.tags.push(value)
      },
      delTag (value) {
        this.form.tags.splice(this.form.tags.indexOf(value), 1)
      },
      addSource (value) {
        this.form.sources.push(value)
      },
      delSource (value) {
        this.form.sources.splice(this.form.sources.indexOf(value), 1)
      },
      querySearch (queryString, cb) {
        api.getProblemTagList().then(res => {
          let tagList = []
          for (let tag of res.data.data) {
            tagList.push({value: tag.name})
          }
          cb(tagList)
        }).catch(() => {})
      },
      submit () {
        this.$refs['form'].validate((valid) => {
          if (valid) {
            this.loading = true
            let params = Object.assign(this.form, {contest_id: this.contestID})
            api.extractQuestion(params).then(res => {
              const data = res.data.data
              this.$message(`Successful import ${data.number}`)
              this.loading = false
              this.$emit('on-change')
            }, () => {
              this.loading = false
            })
          }
        })
      }
    }
  }
</script>

<style scoped>

</style>
