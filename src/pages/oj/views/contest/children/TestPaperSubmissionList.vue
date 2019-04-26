<template>
  <Panel>
    <div slot="title">Test Paper Submission List</div>
    <Table
      :columns="columns"
      :data="submissions"
      :loading="loading"
      no-data-text="No Submission">
    </Table>
    <Pagination
      :page-size="limit"
      :total="total"
      @on-change="getSubmissions"
      key="page">
    </Pagination>
  </Panel>
</template>

<script>
  import { mapGetters } from 'vuex'
  import api from '@oj/api'
  import time from '@/utils/time'
  import Pagination from '@/pages/oj/components/Pagination'
  import { GRADE_STATUS_REVERSE } from '@/utils/constants'

  export default {
    name: 'TestPaperSubmissionList',
    components: {
      Pagination
    },
    data () {
      return {
        total: 30,
        limit: 12,
        loading: false,
        submissions: [],
        regradeColumnsVisible: false,
        columns: [
          {
            title: 'Username',
            align: 'center',
            render: (h, {row}) => {
              return h('a', {
                style: {
                  display: 'inline-block',
                  'max-width': '150px'
                },
                on: {
                  click: () => {
                    this.$router.push({
                      name: 'user-home',
                      query: {username: row.username}
                    })
                  }
                }
              }, row.username)
            }
          },
          {
            title: 'When',
            align: 'center',
            render: (h, {row}) => {
              return h('span', time.utcToLocal(row.create_time))
            }
          },
          {
            title: 'Status',
            align: 'center',
            render: (h, {row}) => {
              return h('Tag', {
                props: {
                  color: GRADE_STATUS_REVERSE[row.status].color
                }
              }, GRADE_STATUS_REVERSE[row.status].name)
            }
          },
          {
            title: 'Score',
            align: 'center',
            key: 'score'
          }
        ]
      }
    },
    mounted () {
      this.getSubmissions()
    },
    methods: {
      getSubmissions (page = 1) {
        this.loading = true
        let params = {
          offset: (page - 1) * this.limit,
          limit: this.limit,
          contest_id: this.contestID
        }
        api.getTestPaperSubmissionList(params).then(res => {
          let data = res.data.data
          for (let v of data.results) {
            v.loading = false
          }
          this.total = data.total
          this.submissions = data.results
          this.adjustRegradeColumn()
          this.loading = false
        })
      },
      adjustRegradeColumn () {
        if (!this.isContestAdmin || this.regradeColumnsVisible) {
          return
        }
        const regradeColumn = {
          title: 'Option',
          fixed: 'right',
          align: 'center',
          width: 90,
          render: (h, {row, index}) => {
            return h('Button', {
              props: {
                type: 'primary',
                size: 'small',
                loading: row.loading
              },
              on: {
                click: () => {
                  this.handleRegrade(row.id, index)
                }
              }
            }, 'Regrade')
          }
        }
        this.columns.push(regradeColumn)
        this.regradeColumnsVisible = true
      },
      handleRegrade (id, index) {
        this.submissions[index].loading = true
        api.regrade(id).then(res => {
          this.submissions[index].loading = false
          this.$success('Successfully created a scoring task')
          this.getSubmissions()
        }, () => {
          this.submissions[index].loading = false
        })
      }
    },
    computed: {
      ...mapGetters(['isContestAdmin']),
      contestID () {
        return this.$route.params.contestID
      }
    }
  }
</script>

<style lang="less" scoped>
</style>
