<template>
  <div class="tile is-ancestor">
    <div class="tile is-vertical">
      <div class="tile">
        <div class="tile is-vertical is-parent is-6">
          <article class="tile is-child notification content-article">
            <div class="content">
              <label class="label">Copy the link of your
                <strong>git repo</strong> here.</label>
              <p class="control has-icons-left" v-bind:class="{ 'has-icons-right': gitSuccess }">
                <input class="input is-medium input-bar" v-focus v-model="giturl" v-on:input="checkGitRepoDebounce"
                       type="text" placeholder="Link to git repo ...">
                <span class="icon is-small is-left">
                  <i class="fa fa-git"></i>
                </span>
                <span v-if="gitSuccess" class="icon is-small is-right is-blue">
                  <i class="fa fa-check"></i>
                </span>
              </p>
              <span style="color: red" v-if="gitErrorMsg">Cannot access git repo: {{ gitErrorMsg }}</span>
              <div v-if="gitBranches.length > 0" style="margin-bottom: 1em;">
                <span>Branch:</span>
                <div class="select is-fullwidth">
                  <select v-model="createPipeline.pipeline.repo.selectedbranch">
                    <option v-for="branch in gitBranches" :key="branch" :value="branch">{{ branch }}</option>
                  </select>
                </div>
              </div>
              <p class="control" style="padding-top: 10px;">
                <a class="button is-primary" v-on:click="showCredentialsModal" style="margin-right: 5px;">
                  <span class="icon">
                    <i class="fa fa-key"></i>
                  </span>
                  <span>Add Credentials</span>
                </a>
                <a class="button is-primary" v-on:click="showGitHubWebHookModal">
                  <span class="icon">
                    <i class="fa fa-github"></i>
                  </span>
                  <span>Add GitHub WebHook</span>
                </a>
              </p>
            </div>
          </article>

          <article class="tile is-child notification content-article">
            <div class="content">
              <p class="control">
                <label class="label">Define custom pipeline options here.</label>
                <a class="button is-primary" v-on:click="showPeriodicalPipelineScheduleModal" style="margin-right: 5px;">
                  <span class="icon">
                    <i class="fa fa-calendar"></i>
                  </span>
                  <span>Add Pipeline Trigger</span>
                </a>
                <a class="button is-primary" v-on:click="showSetPipelineWorkerTags" style="margin-right: 5px;">
                  <span class="icon">
                    <i class="fa fa-tags"></i>
                  </span>
                  <span>Set Pipeline Worker Tags</span>
                </a>
                <a class="button is-primary" v-on:click="showCustomOptionsModal">
                  <span class="icon">
                    <i class="fa fa-cogs"></i>
                  </span>
                  <span>Custom Options</span>
                </a>
              </p>
              <span style="color: red" v-if="periodicSchedulesErr">Periodic schedules invalid: {{ periodicSchedulesErrMsg }}</span>
            </div>
          </article>

          <article class="tile is-child notification content-article">
            <div class="content">
              <label class="label">Type the name of your pipeline here.</label>
              <p class="control has-icons-left" v-bind:class="{ 'has-icons-right': pipelineNameSuccess }">
                <input class="input is-medium input-bar" v-model="pipelinename"
                       v-on:input="checkPipelineNameAvailableDebounce" type="text" placeholder="Pipeline name ...">
                <span class="icon is-small is-left">
                  <i class="fa fa-book"></i>
                </span>
                <span v-if="pipelineNameSuccess" class="icon is-small is-right is-blue">
                  <i class="fa fa-check"></i>
                </span>
              </p>
              <span style="color: red" v-if="pipelineErrorMsg">Pipeline Name incorrect: {{ pipelineErrorMsg }}</span>
            </div>
          </article>
        </div>

        <div class="tile is-parent is-4">
          <article class="tile is-child notification content-article box">
            <p class="subtitle">Select pipeline language</p>
            <div class="content" style="display: flex;">
              <div class="pipelinetype" title="Golang" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'golang'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'golang' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/golang.png" class="typeimage">
              </div>
              <div class="pipelinetype" title="Java" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'java'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'java' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/java.png" class="typeimage">
              </div>
              <div class="pipelinetype" title="Python" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'python'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'python' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/python.png" class="typeimage">
              </div>
            </div>
            <div class="content" style="display: flex;">
              <div class="pipelinetype" title="C++" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'cpp'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'cpp' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/cpp.png" class="typeimage">
              </div>
              <div class="pipelinetype" title="Ruby" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'ruby'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'ruby' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/ruby.png" class="typeimage">
              </div>
              <div class="pipelinetype" title="Node.js" v-tippy="{ arrow : true,  animation : 'shift-away'}"
                   v-on:click="createPipeline.pipeline.type = 'nodejs'"
                   v-bind:class="{ pipelinetypeactive: createPipeline.pipeline.type === 'nodejs' }"
                   data-tippy-hideOnClick="false">
                <img src="../../assets/images/nodejs.png" class="typeimage">
              </div>
            </div>
          </article>
        </div>
      </div>

      <div class="tile is-parent">
        <article class="is-child notification content-article">
          <a class="button is-green-button" style="margin-left: 13px;" v-on:click="startCreatePipeline"
             v-bind:class="{ 'is-disabled': !gitSuccess || !pipelineNameSuccess || periodicSchedulesErr }">
            <span class="icon">
              <i class="fa fa-plus"></i>
            </span>
            <span>Create Pipeline</span>
          </a>
        </article>
      </div>

      <div class="tile is-parent is-10">
        <article class="tile is-child notification content-article box">
          <vue-good-table
            :columns="historyColumns"
            :rows="historyRows"
            :pagination-options="{
              enabled: true,
              mode: 'records'
            }"
            :search-options="{enabled: true, placeholder: 'Search ...'}"
            :sort-options="{
              enabled: true,
              initialSortBy: {field: 'created', type: 'desc'}
            }"
            styleClass="table table-grid table-own-bordered">
            <template slot="table-row" slot-scope="props">
              <span v-if="props.column.field === 'pipeline.name'">{{ props.row.pipeline.name }}</span>
              <span v-if="props.column.field === 'status'" class="progress-bar-height">
                <div class="blink" v-if="props.row.statustype === 'running'" style="height: 25px;">
                  <progress-bar class="progress-bar-middle" :type="'info'" :size="'small'" :value="props.row.status" :max="100"
                                :show-label="false"></progress-bar>
                </div>
                <div v-else-if="props.row.statustype === 'success'" style="color: green;">{{ props.row.statustype }}
                </div>
                <div v-else style="color: red;">{{ props.row.statustype }}</div>
              </span>
              <span v-if="props.column.field === 'pipeline.type'">{{ prettifyPipelineType(props.row.pipeline.type) }}</span>
              <span v-if="props.column.field === 'created'" :title="props.row.created" v-tippy="{ arrow : true,  animation : 'shift-away'}">{{
                convertTime(props.row.created) }}
              </span>
              <span v-if="props.column.field === 'output'">
                <a class="button is-green-button is-small" @click="showStatusOutputModal(props.row.output)">
                    <span class="icon">
                      <i class="fa fa-align-justify"></i>
                    </span>
                  <span>Show Output</span>
                </a>
              </span>
            </template>
            <div slot="emptystate" class="empty-table-text">
              No pipelines found in database.
            </div>
          </vue-good-table>
        </article>
      </div>
    </div>

    <!-- Credentials modal -->
    <modal :visible="gitCredentialsModal" class="modal-z-index" @close="close">
      <div class="box credentials-modal">
        <div class="block credentials-modal-content">
          <collapse accordion is-fullwidth>
            <collapse-item title="Basic Authentication" selected>
              <div class="credentials-modal-content">
                <label class="label" style="text-align: left;">Add credentials for basic authentication:</label>
                <p class="control has-icons-left" style="padding-bottom: 5px;">
                  <input class="input is-medium input-bar" v-focus type="text"
                         v-model="createPipeline.pipeline.repo.user" placeholder="Username">
                  <span class="icon is-small is-left">
                    <i class="fa fa-user-circle"></i>
                  </span>
                </p>
                <p class="control has-icons-left">
                  <input class="input is-medium input-bar" type="password"
                         v-model="createPipeline.pipeline.repo.password" placeholder="Password">
                  <span class="icon is-small is-left">
                    <i class="fa fa-lock"></i>
                  </span>
                </p>
              </div>
            </collapse-item>
            <collapse-item title="SSH Key">
              <label class="label" style="text-align: left;">Instead of using basic authentication, provide a pem
                encoded private key.</label>
              <div class="block credentials-modal-content">
                <p class="control">
                  <textarea class="textarea input-bar" v-model="createPipeline.pipeline.repo.privatekey.key"></textarea>
                </p>
              </div>
              <h2 class="separater">
                <span class="span">Additional:</span>
              </h2>
              <p class="control has-icons-left" style="padding-bottom: 5px;">
                <input class="input is-medium input-bar" v-focus type="text"
                       v-model="createPipeline.pipeline.repo.privatekey.username" placeholder="Username">
                <span class="icon is-small is-left">
                  <i class="fa fa-user-circle"></i>
                </span>
              </p>
              <p class="control has-icons-left">
                <input class="input is-medium input-bar" type="password"
                       v-model="createPipeline.pipeline.repo.privatekey.password" placeholder="Password">
                <span class="icon is-small is-left">
                  <i class="fa fa-lock"></i>
                </span>
              </p>
            </collapse-item>
          </collapse>
          <div class="modal-footer">
            <div style="float: left;">
              <button class="button is-primary" v-on:click="close">Add Credentials</button>
            </div>
            <div style="float: right;">
              <button class="button is-danger" v-on:click="cancel">Cancel</button>
            </div>
          </div>
        </div>
      </div>
    </modal>

    <!-- GitHub WebHook modal -->
    <modal :visible="gitWebHookModal" class="modal-z-index" @close="close">
      <div class="box credentials-modal">
        <div class="block credentials-modal-content">
          <collapse accordion is-fullwidth>
            <collapse-item title="GitHub Token" selected>
              <div class="credentials-modal-content">
                <label class="label" style="text-align: left;">Add GitHub token with repo permissions:</label>
                <p class="control has-icons-left" style="padding-bottom: 5px;">
                  <input class="input is-medium input-bar" v-focus type="text" v-model="createPipeline.githubtoken"
                         placeholder="Token">
                  <span class="icon is-small is-left">
                    <i class="fa fa-github"></i>
                  </span>
                </p>
              </div>
            </collapse-item>
          </collapse>
          <div class="modal-footer">
            <div style="float: left;">
              <button class="button is-primary" v-on:click="close">Add Token</button>
            </div>
            <div style="float: right;">
              <button class="button is-danger" v-on:click="cancel">Cancel</button>
            </div>
          </div>
        </div>
      </div>
    </modal>

    <!-- periodical pipeline schedule modal -->
    <modal :visible="periodicalPipelineScheduleModal" class="modal-z-index" @close="close">
      <div class="box credentials-modal">
        <div class="block credentials-modal-content">
          <collapse accordion is-fullwidth>
            <collapse-item title="Start pipeline periodically:" selected>
              <div class="credentials-modal-content">
                <p class="control">
                  <textarea class="textarea input-bar" v-model="periodicSchedules"></textarea>
                </p>
              </div>
              <label class="label" style="test-align: left;">
                Use the standard cron syntax. For example to start the pipeline every half hour:
                <br/>0 30 * * * *<br/>
                Please see <a href="https://godoc.org/github.com/robfig/cron" target="_blank">here</a> for more
                information.
              </label>
            </collapse-item>
          </collapse>
          <div class="modal-footer">
            <div style="float: left;">
              <button class="button is-primary" v-on:click="addPeriodicalSchedules">Confirm</button>
            </div>
            <div style="float: right;">
              <button class="button is-danger" v-on:click="cancel">Cancel</button>
            </div>
          </div>
        </div>
      </div>
    </modal>

    <!-- set pipeline worker tags modal -->
    <modal :visible="setPipelineWorkerTagsModal" class="modal-z-index" @close="close">
      <div class="box credentials-modal">
        <div class="block credentials-modal-content">
          <collapse accordion is-fullwidth>
            <collapse-item title="Set pipeline worker tags:" selected>
              <div class="credentials-modal-content">
                <message :direction="'down'" style="padding-bottom: 10px;"
                         message="
                          Set custom worker tags to this pipeline which will be evaluated during pipeline execution.
                          WARNING: This pipeline will only be scheduled for worker with the same matching subset of custom tags."
                         :duration="0">
                </message>
                <p class="control has-icons-left" style="padding-bottom: 5px;">
                  <vue-tags-input :tags="selectedPipelineWorkerTags"
                                  v-model="currentTag"
                                  class="input is-medium input-bar"
                                  @tags-changed="newTags => selectedPipelineWorkerTags = newTags">
                  </vue-tags-input>
                  <span class="icon is-small is-left">
                    <i class="fa fa-tags"></i>
                  </span>
                </p>
              </div>
            </collapse-item>
          </collapse>
          <div class="modal-footer">
            <div style="float: left;">
              <button class="button is-primary" v-on:click="setPipelineWorkerTags">Confirm</button>
            </div>
            <div style="float: right;">
              <button class="button is-danger" v-on:click="cancel">Cancel</button>
            </div>
          </div>
        </div>
      </div>
    </modal>

    <!-- pipeline custom options -->
    <modal :visible="customOptionsModal" class="modal-z-index" @close="close">
      <div class="box credentials-modal">
        <div class="block credentials-modal-content">
          <collapse accordion is-fullwidth>
            <collapse-item title="Docker options:" selected>
              <div class="credentials-modal-content">
                <message :direction="'down'" style="padding-bottom: 10px;"
                         message="
                           If enabled, Gaia will automatically run this pipeline in a docker container
                           by default."
                         :duration="0">
                </message>
                <p class="control has-icons-left" style="padding-bottom: 5px;">
                  <toggle-button
                    v-model="createPipeline.pipeline.docker"
                    id="pipelinedocker"
                    :color="{checked: '#7DCE94', unchecked: '#82C7EB'}"
                    :labels="{checked: 'On', unchecked: 'Off'}"
                    :sync="true"/>
                </p>
              </div>
            </collapse-item>
          </collapse>
          <div class="modal-footer">
            <div style="float: left;">
              <button class="button is-primary" v-on:click="customOptionsModalConfirm">Confirm</button>
            </div>
            <div style="float: right;">
              <button class="button is-danger" v-on:click="cancel">Cancel</button>
            </div>
          </div>
        </div>
      </div>
    </modal>

    <!-- status output modal -->
    <modal :visible="statusOutputModal" class="modal-z-index" @close="closeStatusModal">
      <div class="box statusModal">
        <div>
          <message :direction="'down'" :message="statusOutputMsg" :duration="0"></message>
        </div>
        <div class="modal-footer">
          <div style="float: right;">
            <button class="button is-danger" @click="closeStatusModal">Close</button>
          </div>
        </div>
      </div>
    </modal>
  </div>
</template>

<script>
import Vue from 'vue'
import { Modal } from 'vue-bulma-modal'
import { Collapse, Item as CollapseItem } from 'vue-bulma-collapse-fixed'
import ProgressBar from 'vue-bulma-progress-bar'
import VueTippy from 'vue-tippy'
import { VueGoodTable } from 'vue-good-table'
import 'vue-good-table/dist/vue-good-table.css'
import moment from 'moment'
import Notification from 'vue-bulma-notification-fixed'
import Message from 'vue-bulma-message-html'
import VueTagsInput from '@johmun/vue-tags-input'
import { ToggleButton } from 'vue-js-toggle-button'

const NotificationComponent = Vue.extend(Notification)
const openNotification = (
  propsData = {
    title: '',
    message: '',
    type: '',
    direction: '',
    duration: 4500,
    container: '.notifications'
  }
) => {
  return new NotificationComponent({
    el: document.createElement('div'),
    propsData
  })
}

Vue.use(VueTippy)

export default {
  data () {
    return {
      gitErrorMsg: '',
      gitSuccess: false,
      gitCredentialsModal: false,
      gitWebHookModal: false,
      periodicalPipelineScheduleModal: false,
      gitBranches: [],
      giturl: '',
      pipelinename: '',
      pipelineNameSuccess: false,
      pipelineErrorMsg: '',
      periodicSchedules: '',
      periodicSchedulesErrMsg: '',
      periodicSchedulesErr: false,
      createPipeline: {
        id: '',
        output: '',
        status: 0,
        created: new Date(),
        githubtoken: '',
        pipeline: {
          name: '',
          type: 'golang',
          perodicschedules: [],
          tags: [],
          repo: {
            url: '',
            user: '',
            password: '',
            selectedbranch: '',
            privatekey: {
              key: '',
              username: '',
              password: ''
            }
          }
        }
      },
      historyColumns: [
        {
          label: 'Name',
          field: 'pipeline.name'
        },
        {
          label: 'Status',
          field: 'status',
          type: 'number'
        },
        {
          label: 'Type',
          field: 'pipeline.type'
        },
        {
          label: 'Creation date',
          field: 'created'
        },
        {
          label: 'Status Output',
          field: 'output'
        }
      ],
      historyRows: [],
      statusOutputModal: false,
      statusOutputMsg: '',
      setPipelineWorkerTagsModal: false,
      selectedPipelineWorkerTags: [],
      currentTag: '',
      customOptionsModal: false
    }
  },

  components: {
    Modal,
    Collapse,
    CollapseItem,
    ProgressBar,
    Message,
    VueTagsInput,
    VueGoodTable,
    ToggleButton
  },

  mounted () {
    // created pipelines history
    this.fetchData()

    // periodically update history dashboard
    var intervalID = setInterval(function () {
      this.fetchData()
    }.bind(this), 3000)

    // Append interval id to store
    this.$store.commit('appendInterval', intervalID)
  },

  destroyed () {
    this.$store.commit('clearIntervals')
  },

  watch: {
    '$route': 'fetchData'
  },

  methods: {
    fetchData () {
      this.$http
        .get('/api/v1/pipeline/created', { params: { hideProgressBar: true } })
        .then(response => {
          if (response.data) {
            this.historyRows = response.data

            // Add extra output messages if the pipeline is in creation status.
            for (var i = 0; i < this.historyRows.length; i++) {
              if (this.historyRows[i].statustype === 'running' || this.historyRows[i].statustype === 'success') {
                if (this.historyRows[i].status >= 25) {
                  this.historyRows[i].output = 'Git repository has been cloned.\n'
                  this.historyRows[i].output += 'Starting build process. This may take some time...\n'
                }

                if (this.historyRows[i].status >= 50) {
                  this.historyRows[i].output += 'Pipeline has been successfully compiled.\n'
                  this.historyRows[i].output += 'Verifying pipeline...\n'
                }

                if (this.historyRows[i].status >= 75) {
                  this.historyRows[i].output += 'Pipeline has been verified.\n'
                  this.historyRows[i].output += 'Finalizing creation process...\n'
                }

                if (this.historyRows[i].status === 100) {
                  this.historyRows[i].output += 'Copy was successful.\n'
                  this.historyRows[i].output += 'Finished.\n'
                }
              }
            }
          }
        })
        .catch((error) => {
          this.$store.commit('clearIntervals')
          this.$onError(error)
        })
    },

    checkGitRepoDebounce: Vue._.debounce(function () {
      this.checkGitRepo()
    }, 500),

    checkGitRepo () {
      if (this.giturl === '') {
        return
      }

      // copy giturl into our struct
      this.createPipeline.pipeline.repo.url = this.giturl

      // Reset last fetches
      this.gitBranches = []
      this.createPipeline.pipeline.repo.selectedbranch = ''
      this.gitSuccess = false

      this.$http
        .post('/api/v1/pipeline/gitlsremote', this.createPipeline.pipeline.repo)
        .then(response => {
          // Reset error message before
          this.gitErrorMsg = ''
          this.gitSuccess = true

          // Get branches and set to master if available
          this.gitBranches = response.data
          for (var i = 0; i < this.gitBranches.length; i++) {
            if (this.gitBranches[i] === 'refs/heads/master') {
              this.createPipeline.pipeline.repo.selectedbranch = this.gitBranches[i]
            }
          }

          // if we cannot find master
          if (
            !this.createPipeline.pipeline.repo.selectedbranch &&
              this.gitBranches.length > 0
          ) {
            this.createPipeline.pipeline.repo.selectedbranch = this.gitBranches[0]
          }
        })
        .catch(error => {
          if (error.response) {
            // Add error message
            this.gitErrorMsg = error.response.data
          } else {
            this.$onError(error)
          }
        })
    },

    checkPipelineNameAvailableDebounce: Vue._.debounce(function () {
      this.checkPipelineNameAvailable()
    }, 500),

    checkPipelineNameAvailable () {
      // copy pipeline name into struct
      this.createPipeline.pipeline.name = this.pipelinename

      // Request for availability
      this.$http
        .get('/api/v1/pipeline/name', { params: { name: this.pipelinename } })
        .then(response => {
          // pipeline name valid and available
          this.pipelineErrorMsg = ''
          this.pipelineNameSuccess = true
        })
        .catch(error => {
          if (error.response) {
            this.pipelineErrorMsg = error.response.data
            this.pipelineNameSuccess = false
          } else {
            this.$onError(error)
          }
        })
    },

    startCreatePipeline () {
      // Copy related information into the create pipeline struct
      this.createPipeline.pipeline.repo.url = this.giturl
      this.createPipeline.pipeline.name = this.pipelinename

      // Start the create pipeline process in the backend
      this.$http
        .post('/api/v1/pipeline', this.createPipeline)
        .then(response => {
          openNotification({
            title: 'Pipeline created',
            message: `Pipeline "${this.createPipeline.pipeline.name}" has been created successfully.`,
            type: 'success'
          })
          // Run fetchData to see the pipeline in our history table
          this.fetchData()
        })
        .catch((error) => {
          this.$onError(error)
        })
    },

    convertTime (time) {
      return moment(time).fromNow()
    },

    close () {
      this.checkGitRepo()
      this.gitCredentialsModal = false
      this.gitWebHookModal = false
      this.periodicalPipelineScheduleModal = false
      this.setPipelineWorkerTagsModal = false
      this.customOptionsModal = false
      this.$emit('close')
    },

    cancel () {
      // cancel means reset all stuff
      this.createPipeline.pipeline.repo.user = ''
      this.createPipeline.pipeline.repo.password = ''
      this.createPipeline.pipeline.repo.privatekey.key = ''
      this.createPipeline.pipeline.repo.privatekey.username = ''
      this.createPipeline.pipeline.repo.privatekey.password = ''
      this.periodicSchedules = ''
      this.selectedPipelineWorkerTags = []
      this.createPipeline.pipeline.docker = false

      this.close()
    },

    showCredentialsModal () {
      this.gitCredentialsModal = true
    },

    showGitHubWebHookModal () {
      this.gitWebHookModal = true
    },

    showCustomOptionsModal () {
      this.customOptionsModal = true
    },

    showPeriodicalPipelineScheduleModal () {
      this.periodicSchedules = ''
      if (typeof this.createPipeline.pipeline.periodicschedules !== 'undefined') {
        for (let i = 0; i < this.createPipeline.pipeline.periodicschedules.length; i++) {
          this.periodicSchedules += this.createPipeline.pipeline.periodicschedules[i]
        }
      }
      this.periodicalPipelineScheduleModal = true
    },

    addPeriodicalSchedules () {
      // Reset previous errors if there were some.
      this.periodicSchedulesErr = false

      // Split string by line breaks.
      this.createPipeline.pipeline.periodicschedules = this.periodicSchedules.split('\n')

      // Check if periodic schedule entries are valid.
      this.$http
        .post('/api/v1/pipeline/periodicschedules', this.createPipeline.pipeline.periodicschedules)
        .then(response => {
          openNotification({
            title: 'Periodic schedules valid',
            message: `All defined periodic schedules are valid!`,
            type: 'success'
          })
        })
        .catch((error) => {
          this.periodicSchedulesErr = true
          this.periodicSchedulesErrMsg = error.response.data
        })

      this.close()
    },

    setPipelineWorkerTags () {
      // Reset old tags
      this.createPipeline.pipeline.tags = []

      // Add tags to create pipeline object
      for (let i = 0; i < this.selectedPipelineWorkerTags.length; i++) {
        this.createPipeline.pipeline.tags.push(this.selectedPipelineWorkerTags[i].text)
      }

      // set current tag in case the user forgot to press enter
      if (this.currentTag !== '') {
        this.createPipeline.pipeline.tags.push(this.currentTag)
        this.currentTag = ''
      }

      this.close()
    },

    customOptionsModalConfirm () {
      this.close()
    },

    showSetPipelineWorkerTags () {
      this.selectedPipelineWorkerTags = []
      for (let i = 0; i < this.createPipeline.pipeline.tags.length; i++) {
        this.selectedPipelineWorkerTags.push({ text: this.createPipeline.pipeline.tags[i], tiClasses: ['ti-valid'] })
      }
      this.setPipelineWorkerTagsModal = true
    },

    showStatusOutputModal (msg) {
      if (!msg) {
        msg = 'No output found.'
      }

      // LF does not work for HTML. Replace with <br />
      msg = msg.replace(/\n/g, '<br />')

      this.statusOutputMsg = msg
      this.statusOutputModal = true
    },

    closeStatusModal () {
      this.statusOutputModal = false
      this.statusOutputMsg = ''
      this.$emit('close')
    },

    prettifyPipelineType (type) {
      if (type === 'cpp') {
        return 'C++'
      }
      return type
    }
  }
}
</script>

<style lang="scss">
  .credentials-modal {
    text-align: center;
    background-color: #2a2735;
  }

  .credentials-modal-content {
    margin: auto;
    padding: 10px;
  }

  .dotted-line {
    background-image: linear-gradient(
      to right,
      black 33%,
      rgba(255, 255, 255, 0) 0%
    );
    background-position: bottom;
    background-size: 3px 1px;
    background-repeat: repeat-x;
  }

  .separater {
    width: 100%;
    text-align: center;
    border-bottom: 1px solid #4da2fc;
    line-height: 0.1em;
    padding-top: 15px;
    margin: 10px 0 20px;
  }

  .separater .span {
    background: black;
    color: whitesmoke;
    padding: 0 10px;
  }

  .modal-footer {
    height: 35px;
    padding-top: 15px;
  }

  .pipelinetype {
    height: 80px;
    width: 80px;
    border: 1px solid;
    color: black;
    margin: 0 5px;
    box-shadow: 4px 4px 4px 4px rgba(0, 0, 0, 0.2),
    0 1px 1px 0 rgba(0, 0, 0, 0.19);
  }

  .pipelinetype:hover {
    -moz-transform: scale(1.1);
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }

  .pipelinetypeactive {
    color: #4da2fc;
    border: 2px solid;
  }

  .typeimage {
    display: block;
    margin: 0 auto;
    height: 100%;
  }

  .typeimagenotyetsupported {
    opacity: 0.5;
  }

  .blink {
    animation: blink 700ms infinite alternate;
  }

  @keyframes blink {
    from {
      opacity: 1;
    }
    to {
      opacity: 0.2;
    }
  }

  .message-header {
    background-color: #4da2fc;
  }

  .message-body {
    background-color: #19191b;
    border-radius: 0px;
    border: none;
    color: whitesmoke;
  }

  .statusModal {
    width: 100%;
    background-color: rgb(42, 38, 53);
  }

  .ti-input {
    background-color: #19191b;
    color: white;
    border: none !important;
    width: 100%;
    height: 100% !important;
    padding: 0px !important;
    caret-color: whitesmoke;
  }

  .ti-new-tag-input-wrapper {
    height: 30px !important;
    border: none !important;
    margin: auto !important;
    padding: inherit !important;
  }

  .ti-new-tag-input {
    background-color: #19191b !important;
    font-size: 1.25rem;
    color: whitesmoke !important;
  }

  .ti-new-tag-input::-webkit-input-placeholder {
    color: #8c91a0;
    text-shadow: none;
    -webkit-text-fill-color: initial;
  }

  .ti-new-tag-input::-moz-placeholder {
    color: #8c91a0;
    text-shadow: none;
    opacity: 1;
  }

  .vue-tags-input {
    background-color: #19191b !important;
    height: auto !important;
  }
</style>
