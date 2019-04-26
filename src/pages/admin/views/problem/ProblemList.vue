<template>
  <div class="view">
    <Panel :title="contestId ? 'Contest Problem List' : 'Problem List'">
      <el-row :gutter="20" slot="header">
        <el-col :span="12">
          <el-select v-model="type">
            <el-option :label="$t('m.ProgrammingQuestion')" :value="QUESTION_TYPE.CODING"></el-option>
            <el-option :label="$t('m.FormQuestion')" :value="QUESTION_TYPE.FORM"></el-option>
          </el-select>
        </el-col>
        <el-col :span="12">
          <el-input
            placeholder="Keywords"
            prefix-icon="el-icon-search"
            v-model="keyword">
          </el-input>
        </el-col>
      </el-row>
      <el-table
        v-loading="loading"
        element-loading-text="loading"
        ref="table"
        :data="problemList"
        @row-dblclick="handleDblclick"
        style="width: 100%">
        <el-table-column
          width="100"
          prop="id"
          label="ID">
        </el-table-column>
        <el-table-column
          width="150"
          label="Display ID"
          key="_id"
          prop="_id"
          v-if="isCodingType()">
          <template slot-scope="{row}">
            <span v-show="!row.isEditing">{{row._id}}</span>
            <el-input v-show="row.isEditing" v-model="row._id"
                      @keyup.enter.native="handleInlineEdit(row)">
            </el-input>
          </template>
        </el-table-column>
        <el-table-column
          prop="title"
          :label="isCodingType() ? 'Title' : 'Summary'">
          <template slot-scope="{row}">
            <span v-show="!row.isEditing">{{row.title}}</span>
            <el-input v-show="row.isEditing" v-model="row.title"
                      @keyup.enter.native="handleInlineEdit(row)">
            </el-input>
          </template>
        </el-table-column>
        <el-table-column
          prop="created_by.username"
          label="Author">
        </el-table-column>
        <el-table-column
          width="200"
          prop="create_time"
          label="Create Time">
          <template slot-scope="scope">
            {{scope.row.create_time | localtime }}
          </template>
        </el-table-column>
        <el-table-column
          width="100"
          prop="visible"
          label="Visible"
          key="visible"
          v-if="isCodingType()">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.visible"
                       active-text=""
                       inactive-text=""
                       @change="updateProblem(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="Operation"
          width="250">
          <div slot-scope="scope">
            <icon-btn name="Edit" icon="edit" @click.native="goEdit(scope.row.id)"></icon-btn>
            <icon-btn icon="clone" name="Make Public" v-if="isCodingType() && contestId"
                      @click.native="makeContestProblemPublic(scope.row.id)"></icon-btn>
            <icon-btn icon="download" name="Download TestCase" v-if="isCodingType()"
                      @click.native="downloadTestCase(scope.row.id)"></icon-btn>
            <icon-btn icon="trash" name="Delete Problem"
                      @click.native="deleteProblem(scope.row.id)"></icon-btn>
          </div>
        </el-table-column>
      </el-table>
      <div class="panel-options">
        <el-button size="small" type="primary" v-if="isCodingType() || !contestId"
                   @click="goCreateProblem" icon="el-icon-plus">
          Create
        </el-button>
        <el-button type="primary" v-if="!isCodingType() && contestId"
                   size="small" icon="el-icon-plus"
                   @click="extractQuestionDialogVisible = true">
          Extract Question
        </el-button>
        <el-button v-if="contestId" type="primary"
                   size="small" icon="el-icon-plus"
                   @click="addProblemDialogVisible = true">
          Add From Public Problem
        </el-button>
        <el-pagination
          class="page"
          layout="prev, pager, next"
          @current-change="currentChange"
          :page-size="pageSize"
          :total="total">
        </el-pagination>
      </div>
    </Panel>
    <el-dialog title="Sure to update the problem? "
               width="20%"
               :visible.sync="InlineEditDialogVisible"
               @close-on-click-modal="false">
      <div>
        <p>DisplayID: {{currentRow._id}}</p>
        <p>Title: {{currentRow.title}}</p>
      </div>
      <span slot="footer">
        <cancel @click.native="InlineEditDialogVisible = false; getProblemList(currentPage)"></cancel>
        <save @click.native="updateProblem(currentRow)"></save>
      </span>
    </el-dialog>
    <el-dialog title="Add Contest Problem"
               v-if="contestId"
               width="80%"
               :visible.sync="addProblemDialogVisible"
               @close-on-click-modal="false">
      <add-problem-component :contestID="contestId" :type="type" @on-change="getProblemList"></add-problem-component>
    </el-dialog>
    <el-dialog
      title="Extract Contest Question"
      v-if="contestId"
      :visible.sync="extractQuestionDialogVisible"
      @close-on-click-modal="false">
      <extract-question :contestID="contestId" @on-change="getProblemList"></extract-question>
    </el-dialog>
  </div>
</template>

<script>
  import api from '../../api.js'
  import utils from '@/utils/utils'
  import AddProblemComponent from './AddPublicProblem.vue'
  import { QUESTION_TYPE } from '../../../../utils/constants'
  import ExtractQuestion from './ExtractQuestion'

  export default {
    name: 'ProblemList',
    components: {
      ExtractQuestion,
      AddProblemComponent
    },
    data () {
      return {
        pageSize: 10,
        total: 0,
        problemList: [],
        keyword: '',
        type: QUESTION_TYPE.CODING,
        loading: false,
        currentPage: 1,
        routeName: '',
        contestId: '',
        // for make public use
        currentProblemID: '',
        currentRow: {},
        InlineEditDialogVisible: false,
        makePublicDialogVisible: false,
        addProblemDialogVisible: false,
        extractQuestionDialogVisible: false,
        QUESTION_TYPE: QUESTION_TYPE
      }
    },
    mounted () {
      this.routeName = this.$route.name
      this.contestId = this.$route.params.contestId
      this.getProblemList(this.currentPage)
    },
    methods: {
      isCodingType () {
        return this.type === QUESTION_TYPE.CODING
      },
      handleDblclick (row) {
        row.isEditing = true
      },
      goEdit (problemId) {
        if (this.isCodingType()) {
          if (this.routeName === 'problem-list') {
            this.$router.push({name: 'edit-problem', params: {problemId}})
          } else if (this.routeName === 'contest-problem-list') {
            this.$router.push({name: 'edit-contest-problem', params: {problemId: problemId, contestId: this.contestId}})
          }
        } else {
          this.$router.push({name: 'edit-question', params: {problemId}})
        }
      },
      goCreateProblem () {
        if (this.isCodingType()) {
          if (this.routeName === 'problem-list') {
            this.$router.push({name: 'create-problem'})
          } else if (this.routeName === 'contest-problem-list') {
            this.$router.push({name: 'create-contest-problem', params: {contestId: this.contestId}})
          }
        } else {
          this.$router.push({name: 'create-question'})
        }
      },
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getProblemList(page)
      },
      getProblemList (page = 1) {
        this.loading = true
        let funcName = this.isCodingType() ? (
          this.routeName === 'problem-list' ? 'getProblemList' : 'getContestProblemList') : (
            this.routeName === 'problem-list' ? 'getQuestionList' : 'getContestQuestionList')
        let params = {
          limit: this.pageSize,
          offset: (page - 1) * this.pageSize,
          keyword: this.keyword,
          contest_id: this.contestId
        }
        api[funcName](params).then(res => {
          this.total = res.data.data.total
          for (let problem of res.data.data.results) {
            problem.isEditing = false
          }
          this.problemList = res.data.data.results
          this.loading = false
        }, res => {
          this.loading = false
        })
      },
      deleteProblem (id) {
        this.$confirm(
          'Sure to delete this problem? The associated submissions will be deleted as well.',
          'Delete Problem',
          { type: 'warning' }).then(() => {
            let funcName = this.isCodingType() ? (
              this.routeName === 'problem-list' ? 'deleteProblem' : 'deleteContestProblem') : (
                this.routeName === 'problem-list' ? 'deleteQuestion' : 'deleteContestQuestion')
            api[funcName](id).then(() => [
              this.getProblemList(this.currentPage - 1)
            ]).catch(() => {})
          }, () => {})
      },
      makeContestProblemPublic (problemID) {
        this.$prompt('Please input display id for the public problem', 'confirm').then(({value}) => {
          api.makeContestProblemPublic({id: problemID, display_id: value}).catch()
        }, () => {
        })
      },
      updateProblem (row) {
        let data = Object.assign({}, row)
        let funcName = ''
        if (this.contestId) {
          data.contest_id = this.contestId
          funcName = 'editContestProblem'
        } else {
          funcName = 'editProblem'
        }
        api[funcName](data).then(res => {
          this.InlineEditDialogVisible = false
          this.getProblemList(this.currentPage)
        }).catch(() => {
          this.InlineEditDialogVisible = false
        })
      },
      handleInlineEdit (row) {
        this.currentRow = row
        this.InlineEditDialogVisible = true
      },
      downloadTestCase (problemID) {
        let url = '/admin/test_case?problem_id=' + problemID
        utils.downloadFile(url)
      }
    },
    watch: {
      '$route' (newVal, oldVal) {
        this.contestId = newVal.params.contestId
        this.routeName = newVal.name
        this.getProblemList(this.currentPage)
      },
      'keyword' () {
        this.currentChange()
      },
      'type' () {
        this.currentChange()
      }
    }
  }
</script>

<style scoped lang="less">
</style>
