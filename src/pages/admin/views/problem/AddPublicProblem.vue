<template>
  <div>
    <el-input
      v-model="keyword"
      placeholder="Keywords"
      prefix-icon="el-icon-search">
    </el-input>
    <el-table :data="problems" v-loading="loading">
      <el-table-column
        label="ID"
        width="100"
        prop="id">
      </el-table-column>
      <el-table-column
        label="DisplayID"
        width="200"
        prop="_id"
        v-if="isCodingType()">
      </el-table-column>
      <el-table-column
        :label="isCodingType() ? 'Title' : 'Summary'"
        prop="title">
      </el-table-column>
      <el-table-column
        label="option"
        align="center"
        width="100"
        fixed="right">
        <template slot-scope="{row}">
          <icon-btn
            :disabled="row.disabled"
            @click.native="handleAddProblem(row.id, row.$index)"
            icon="plus"
            name="Add the problem"></icon-btn>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="page"
      layout="prev, pager, next"
      @current-change="getPublicProblem"
      :page-size="limit"
      :total="total">
    </el-pagination>
  </div>
</template>
<script>
  import api from '@admin/api'
  import { QUESTION_TYPE } from '../../../../utils/constants'

  export default {
    name: 'add-problem-from-public',
    props: ['contestID', 'type'],
    data () {
      return {
        page: 1,
        limit: 10,
        total: 0,
        loading: false,
        problems: [],
        contest: {},
        keyword: '',
        QUESTION_TYPE: QUESTION_TYPE
      }
    },
    mounted () {
      api.getContest(this.contestID).then(res => {
        this.contest = res.data.data
        this.getPublicProblem()
      }).catch(() => {
      })
    },
    methods: {
      isCodingType () {
        return this.type === QUESTION_TYPE.CODING
      },
      getPublicProblem (page) {
        this.loading = true
        let params = {
          keyword: this.keyword,
          offset: (page - 1) * this.limit,
          limit: this.limit
        }
        let funcName = this.isCodingType() ? 'getProblemList' : 'getQuestionList'
        if (this.isCodingType()) {
          params.rule_type = this.contest.rule_type
        }
        api[funcName](params).then(res => {
          this.loading = false
          this.total = res.data.data.total
          this.problems = res.data.data.results
        }).catch(() => {})
      },
      handleAddProblem (problemID, index) {
        if (this.isCodingType()) {
          this.$prompt('Please input display id for the contest problem', 'confirm').then(({value}) => {
            let data = {
              problem_id: problemID,
              contest_id: this.contestID,
              display_id: value
            }
            api.addProblemFromPublic(data).then(() => {
              this.problems[index].disabled = true
              this.$emit('on-change')
            }, () => {})
          }, () => {})
        } else {
          let data = {
            question_id: problemID,
            contest_id: this.contestID
          }
          api.addContestQuestion(data).then(() => {
            this.problems[index].disabled = true
            this.$emit('on-change')
          })
        }
      }
    },
    watch: {
      'keyword' () {
        this.getPublicProblem(this.page)
      },
      'type' () {
        this.getPublicProblem(this.page)
      }
    }
  }
</script>
<style scoped>
  .page {
    margin-top: 20px;
    text-align: right
  }

</style>
