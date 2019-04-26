<template>
  <div class="flex-container">
    <div id="problem-main">
      <Alert show-icon type="error" v-if="contestEnded">Contest has ended</Alert>
      <Card
        v-for="(question, index) in questions"
        :key="question.id"
        :padding="20"
        shadow>
        <div id="problem-content" class="markdown-body" v-katex>
          <p class="content" v-html=question.description></p>
        </div>
        <vue-form-generator
          :schema="question.schema"
          :isNewModel="true"
          :model="question.content"
          class="question-form">
        </vue-form-generator>
      </Card>
    </div>

    <div id="right-column">
      <VerticalMenu @on-click="handleRoute">
        <VerticalMenu-item :route="{name: 'contest-problem-list', params: {contestID: contestID}}">
          Problems
        </VerticalMenu-item>
        <VerticalMenu-item :route="{name: 'contest-announcement-list', params: {contestID: contestID}}">
          Announcements
        </VerticalMenu-item>
        <VerticalMenu-item :route="{name: 'contest-details', params: {contestID: contestID}}">
          View Contest
        </VerticalMenu-item>
      </VerticalMenu>

      <Card id="operation">
        <ul>
          <li>
            <span>Status: </span>
            <Tag :color="GRADE_STATUS_REVERSE[submission.status].color">
              {{GRADE_STATUS_REVERSE[submission.status].name}}
            </Tag>
          </li>
          <li v-if="submission.score">
            <span>Score: </span>
            <span>{{submission.score}}</span>
          </li>
          <li v-if="submission.create_time">
            <span>When: </span>
            <span>{{ submission.create_time | localtime }}</span>
          </li>
        </ul>
        <Row v-if="captchaRequired">
          <Col :span="12">
            <img :src="captchaSrc" @click="getCaptchaSrc" alt="captcha"/>
          </Col>
          <Col :span="12">
            <Input v-model="captchaCode"></Input>
          </Col>
        </Row>
        <Row>
          <Col :span="24">
            <Button
              type="primary"
              :disabled="submission.status !== GRADE_STATUS.NO_SUBMITTED"
              @click="submit"
              icon="edit"
              long>
              Submit
            </Button>
          </Col>
        </Row>
      </Card>
    </div>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import VueFormGenerator from 'vue-form-generator'
  import 'vue-form-generator/dist/vfg.css'
  import types from '../../../../store/types'
  import api from '@oj/api'
  import { FormMixin } from '@oj/components/mixins'
  import { CONTEST_STATUS, GRADE_STATUS, GRADE_STATUS_REVERSE } from '../../../../utils/constants'

  export default {
    name: 'TestPaper',
    components: {
      'vue-form-generator': VueFormGenerator.component
    },
    mixins: [FormMixin],
    data () {
      return {
        contestID: '',
        questions: [],
        submission: {
          status: GRADE_STATUS.NO_SUBMITTED
        },
        captchaRequired: false,
        captchaCode: '',
        GRADE_STATUS: GRADE_STATUS,
        GRADE_STATUS_REVERSE: GRADE_STATUS_REVERSE
      }
    },
    watch: {
      '$route' () {
        this.init()
      }
    },
    mounted () {
      this.$store.commit(types.CHANGE_CONTEST_ITEM_VISIBLE, {menu: false})
      this.init()
    },
    beforeRouteLeave (to, from, next) {
      this.$store.commit(types.CHANGE_CONTEST_ITEM_VISIBLE, {menu: true})
      next()
    },
    methods: {
      init () {
        this.$Loading.start()
        this.contestID = this.$route.params.contestID
        api.getTestPaper(this.contestID).then(res => {
          this.questions = res.data.data
          this.questions.forEach((question) => {
            question.content = {}
          })
          this.getSubmissionStatus()
          this.$Loading.finish()
        }, () => {
          this.$Loading.error()
        })
      },
      submit () {
        let content = {}
        this.questions.forEach((question) => {
          if (question.content) {
            content[question.id] = question.content
          }
        })
        let data = {
          contest_id: this.contestID,
          content: content
        }
        if (this.captchaRequired) {
          data.captcha = this.captchaCode
        }
        this.$Modal.confirm({
          title: 'Submit',
          content: 'Make sure all the questions are finished before handing in the paper.',
          onOk: () => {
            api.submitAnswer(data).then(res => {
              this.submission = res.data.data
            }, res => {
              this.getCaptchaSrc()
              if (res.data.data.startsWith('Captcha is required')) {
                this.captchaRequired = true
              }
            })
          }
        })
      },
      getSubmissionStatus () {
        let params = {
          myself: true,
          contest_id: this.contestID
        }
        api.getTestPaperSubmission(params).then(res => {
          this.submission = res.data.data
        })
      },
      handleRoute (route) {
        this.$router.push(route)
      }
    },
    computed: {
      ...mapGetters(['contestStatus']),
      contestEnded () {
        return this.contestStatus === CONTEST_STATUS.ENDED
      }
    }
  }
</script>

<style lang="less">
  .flex-container {
    #problem-main {
      flex: auto;
      margin-right: 18px;

      .ivu-card + .ivu-card {
        margin-top: 20px;
      }
    }
    #right-column {
      flex: none;
      width: 220px;
      position: sticky;
      top: 75px;
    }
  }

  #problem-content {
    p.content {
      font-size: 15px
    }
  }

  #operation {
    margin-top: 20px;

    ul {
      list-style-type: none;

      li {
        margin-bottom: 10px;

        p {
          display: inline-block;
        }

        p:first-child {
          width: 90px;
        }

        p:last-child {
          float: right;
        }
      }
    }

    .captcha-container {
      display: inline-block;
      .captcha-code {
        width: auto;
        margin-top: -20px;
        margin-left: 20px;
      }
    }
  }

  .question-form {
    fieldset {
      border: 0;
    }
  }
</style>
