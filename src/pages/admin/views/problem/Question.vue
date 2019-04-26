<template>
  <Panel :title="title">
    <el-form :model="question" :rules="rules" label-position="top" label-width="70px" ref="form">
      <el-row :gutter="20">
        <el-col :span="16">
          <el-form-item :label="$t('m.Summary')" prop="title" required>
            <el-input :placeholder="$t('m.Summary')" v-model="question.title"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item :label="$t('m.Score')" prop="score" required>
            <el-input :placeholder="$t('m.Score')" v-model="question.score"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="24">
          <el-form-item :label="$t('m.Description')" prop="description" required>
            <Simditor v-model="question.description"></Simditor>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="24">
          <el-form-item :label="$t('m.FormSchema')" prop="schema" required>
            <code-mirror @blur="$refs['form'].validateField('schema')" mode="application/json" v-model="schema"></code-mirror>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row class="schema-review" :gutter="20" v-if="Object.keys(question.schema).length">
        <el-col :span="12">
          <vue-form-generator
            :isNewModel="true"
            :model="question.content"
            :schema="question.schema">
          </vue-form-generator>
        </el-col>
        <el-col :span="12" v-if="Object.keys(question.content).length">
          <pre v-html="prettyJSON(question.content)"></pre>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="24">
          <el-form-item :label="$t('m.GradeScript')" prop="script" required>
            <code-mirror mode="text/x-lua" v-model="question.script"></code-mirror>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-form-item :label="$t('m.Tag')" prop="tags" required>
            <tag-input
              :fetchSuggestions="querySearch"
              :tags="question.tags"
              @addTag="addTag"
              @delTag="delTag"></tag-input>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item :label="$t('m.Difficulty')">
            <el-select :placeholder="$t('m.Difficulty')" class="difficulty-select" size="small" v-model="question.difficulty">
              <el-option label="Low" value="Low"></el-option>
              <el-option label="Mid" value="Mid"></el-option>
              <el-option label="High" value="High"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item :label="$t('m.Source')">
        <el-input :placeholder="$t('m.Source')" v-model="question.source"></el-input>
      </el-form-item>
      <save @click.native="submit()">Save</save>
    </el-form>
  </Panel>
</template>

<script>
  import TagInput from '../../components/TagInput'
  import Simditor from '../../components/Simditor'
  import api from '../../api'
  import CodeMirror from '../../components/CodeMirror'
  import VueFormGenerator from 'vue-form-generator'
  import 'vue-form-generator/dist/vfg.css'

  export default {
    name: 'Question',
    components: {
      CodeMirror,
      Simditor,
      TagInput,
      'vue-form-generator': VueFormGenerator.component
    },
    data () {
      let validateSchema = (rule, value, callback) => {
        try {
          if (this.schema === '') {
            callback(new Error('Form schema is required.'))
          } else {
            this.question.schema = JSON.parse(this.schema)
            callback()
          }
        } catch (e) {
          callback(e)
        }
      }
      return {
        title: '',
        rules: {
          title: {required: true, message: 'Title is required', trigger: 'blur'},
          description: {required: true, message: 'Question Description is required.', trigger: 'blur'},
          schema: {validator: validateSchema},
          script: {required: true, message: 'Grade script is required.', trigger: 'blur'},
          tags: {type: 'array', required: true, message: 'Please add at least one tag', trigger: 'change'},
          score: {type: 'number', min: 1, message: 'Score must be a positive integer', trigger: 'blur'}
        },
        schema: '',
        question: {
          title: '',
          description: '',
          schema: {},
          script: '',
          tags: [],
          source: '',
          difficulty: 'Low',
          score: 1,
          content: {}
        }
      }
    },
    computed: {
      editMode: function () {
        return this.$route.name === 'edit-question'
      }
    },
    mounted () {
      if (this.editMode) {
        api.getQuestion(this.$route.params.problemId).then(res => {
          this.question = res.data.data
          this.schema = JSON.stringify(this.question.schema, undefined, 2)
        })
        this.title = 'Edit Form Question'
      } else {
        this.title = 'Add Form Question'
      }
    },
    methods: {
      addTag (value) {
        this.question.tags.push(value)
      },
      delTag (value) {
        this.question.tags.splice(this.question.tags.indexOf(value), 1)
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
            let funcName = this.editMode ? 'editQuestion' : 'createQuestion'
            api[funcName](this.question).then(res => {
              this.$router.push({name: 'problem-list'})
            }).catch(() => {})
          }
        })
      },
      prettyJSON: function (json) {
        if (json) {
          json = JSON.stringify(json, undefined, 2)
          json = json.replace(/&/g, '&').replace(/</g, '<').replace(/>/g, '>')
          return json.replace(/("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+-]?\d+)?)/g, function (match) {
            let cls = 'number'
            if (/^"/.test(match)) {
              if (/:$/.test(match)) {
                cls = 'key'
              } else {
                cls = 'string'
              }
            } else if (/true|false/.test(match)) {
              cls = 'boolean'
            } else if (/null/.test(match)) {
              cls = 'null'
            }
            return '<span class="' + cls + '">' + match + '</span>'
          })
        }
      }
    }
  }
</script>

<style lang="less">
  .schema-review {
    fieldset {
      border: 0;
    }

    pre {
      overflow: auto;
    }

    pre .string {
      color: #885800;
    }

    pre .number {
      color: blue;
    }

    pre .boolean {
      color: magenta;
    }

    pre .null {
      color: red;
    }

    pre .key {
      color: green;
    }
  }
</style>
