<template>

  <div :class="customClass">

    <PageHeader
      :dgfType="'issues'"
      :noSubtitle="noOperationLink"
      :compact="compact"
      >
      <template v-slot:subtitle>
        <div class="mb-2">
          {{ $t('navigation.from') }} :
          <span v-if="issuesRequest">
            <a :href="issuesRequest" target="blank">
              {{ issuesRequest }}
            </a>
          </span>
          <span v-else>
            <code>{{ operationId }}</code>
          </span>
        </div>
      </template>
      <template v-slot:badge>
        <h4 v-if="issues">
          <b-badge pill variant="primary">
            {{ pagination.totalItems }}
            {{ $t('basics.issues', {list: ''}) }}
          </b-badge>
        </h4>
      </template>
    </PageHeader>

    <b-card
      class="mx-3 text-center border-0"
      >

      <!-- <p><slot name="blockTitle"></slot></p> -->

      <!-- <code>{{itemsSelection}}</code> -->

      <p><slot name="link" class="mb-3"></slot></p>

      <b-row
        align-v="center"
        class="my-3"
        >

        <b-col cols="6" md="5">
          <b-input-group>
            <b-input-group-prepend is-text>
              <b-icon icon="search"></b-icon>
            </b-input-group-prepend>
            <b-form-input
              id="inline-form-input-query-issues"
              :placeholder="$t('actions.searchFor', {target: $t('basics.issue')})"
              v-model="query"
              @input="getIssues(true)"
              >
            </b-form-input>
            <b-input-group-append v-if="query">
              <b-button variant="outline-secondary" @click="resetQuery">
                <b-icon icon="x" aria-hidden="true"></b-icon>
              </b-button>
            </b-input-group-append>
          </b-input-group>
        </b-col>

        <b-col cols="4" md="5"
          v-if="issues && pagination.totalItems > pagination.pageSize"
          class="my-2"
          >
          <b-pagination
            @input="changePagination"
            v-model="pagination.page"
            :total-rows="pagination.totalItems"
            :per-page="pagination.pageSize"
            class="my-0"
            align="center"
            size="sm"
          ></b-pagination>
        </b-col>

        <b-col cols="2" class="text-right">
          <ModerationActionsBtn
            :dgfType="dgfType"
            :endpoint="endpointModeration"
            :itemsSelection="itemsSelection"
            :itemsList="issues && issues.data"
            @responseAction="callbackAction"
            >
          </ModerationActionsBtn>
        </b-col>

      </b-row>

      <b-table
        v-if="issues && !isLoading"
        striped hover responsive
        @sort-changed="changeSorting"
        :small="small"
        :sticky-header="height"
        :items="issues.data"
        :fields="fields"
        :sort-by.sync="pagination.sortBy"
        :sort-desc.sync="pagination.sortDesc"
        >

        <template v-slot:cell(selection)="data">
          <b-form inline class="justify-content-center">
            <b-button
              :disabled="!isAuthenticated"
              @click="changeSelection(data.item)"
              button
              variant="link"
              >
              <b-icon
                :icon="`${isSelected(data.item) ? 'check2-' : ''}square`"
                :variant="`${isSelected(data.item)? 'primary' : ''}`"
                aria-hidden="true"
                >
              </b-icon>
            </b-button>
          </b-form>
        </template>

        <template v-slot:cell(moderation)="row">
          <b-button
            v-if="isAuthenticated"
            size="sm"
            @click="row.toggleDetails" class="mx-2">
            <b-icon :icon="row.detailsShowing ? 'eye-slash-fill' : 'eye-fill' " aria-hidden="true"></b-icon>
          </b-button>
        </template>

        <template v-if="isAuthenticated" v-slot:row-details="row">
          <ModerationRowCard
            :dgfType="dgfType"
            :item="row.item"
          />
        </template>

        <template v-slot:cell(moderation_read)="row">
          <ModerationCheckbox
            :dgfType="dgfType"
            :item="row.item"
            :field="'read'"
            >
          </ModerationCheckbox>
        </template>

        <template v-slot:cell(moderation_suspect)="row">
          <ModerationCheckbox
            :dgfType="dgfType"
            :item="row.item"
            :field="'suspicious'"
            >
          </ModerationCheckbox>
        </template>

        <template v-slot:cell(moderation_deleted)="row">
          <ModerationCheckbox
            :dgfType="dgfType"
            :item="row.item"
            :field="'deleted'"
            >
          </ModerationCheckbox>
        </template>

        <template v-slot:cell(discussion)="data">
          {{ data.item.discussion.length }}
        </template>

        <template v-slot:cell(id)="data">
          <router-link
            class="text-primary"
            :to="`/issues/${data.value}`"
            >
            {{ data.value }}
          </router-link>
        </template>

        <template v-slot:cell(created)="data">
          <i>{{ formatDate(data.value, addTime = false) }}</i>
        </template>

        <template v-slot:cell(title)="data">
          <router-link
            class="text-primary"
            :to="`/issues/${data.item.id}`"
            >
            <span>
              {{ data.item.title }}
            </span>
          </router-link>
        </template>

        <template v-slot:cell(subject)="data">
          <b-button
            variant="outline-primary"
            :to="`/${data.item.subject.class.toLowerCase()}/${data.item.subject.id}`"
            >
            {{ data.item.subject.class.toLowerCase() }}
          </b-button>
        </template>

      </b-table>

      <p v-if="isLoading">
        <b-spinner label="loading"></b-spinner>
      </p>
    </b-card>

  </div>

</template>

<script>
import { mapState, mapGetters } from 'vuex'

import PageHeader from '@/components/ux/PageHeader.vue'

import ModerationRowCard from '@/components/moderation/ModerationRowCard.vue'
import ModerationCheckbox from '@/components/moderation/ModerationCheckbox.vue'
import ModerationActionsBtn from '@/components/moderation/ModerationActionsBtn.vue'

export default {
  name: 'IssuesList',
  components: {
    PageHeader,
    ModerationActionsBtn,
    ModerationCheckbox,
    ModerationRowCard
  },
  props: [
    'height',
    'small',
    'customFields',
    'noOperationLink',
    'compact',
    'customClass'
  ],
  data () {
    return {
      isLoading: false,
      seeRaw: false,
      dgfType: 'issue',
      endpointModeration: 'issues',
      operationId: 'list_issues',
      issues: undefined,
      issuesRequest: undefined,
      itemsSelection: [],
      needsModerationData: false,
      query: undefined,
      pagination: {
        page: 1,
        pageSize: 20,
        sortBy: 'created',
        sortDesc: false
      },
      fields: [
        // 'index',
        { key: 'selection', label: 'selection', stickyColumn: true, isRowHeader: true, sortable: false },
        { key: 'moderation', label: 'Moderation', stickyColumn: true, isRowHeader: true },
        { key: 'moderation_read', label: 'Moderation', stickyColumn: true, isRowHeader: true, sortable: true },
        { key: 'moderation_suspect', label: 'Suspect', stickyColumn: true, isRowHeader: true, sortable: true },
        { key: 'moderation_deleted', label: 'Deleted', stickyColumn: true, isRowHeader: true, sortable: true },
        { key: 'title', label: 'title', stickyColumn: true, isRowHeader: true },
        { key: 'discussion', label: 'number of discussions', sortable: true },
        { key: 'subject', label: 'related to' },
        { key: 'created', label: 'created at', sortable: true },
        'id'
      ]
    }
  },
  created () {
    if (this.customFields) { this.fields = this.customFields }
    this.getIssues()
  },
  watch: {
    async issues (next) {
      if (next && this.needsModerationData) {
        this.issues = await this.appendModerationData(next)
      }
    }
  },
  computed: {
    ...mapState({
      log: (state) => state.log
    }),
    ...mapGetters({
      isAuthenticated: 'oauth/isAuthenticated'
    })
  },
  methods: {
    async appendModerationData (itemObject) {
      if (this.isAuthenticated) {
        const newData = await Promise.all(itemObject.data.map(async (obj) => {
          const itemStatus = await this.$MODERATIONcli.getModeration(obj.id)
          const consolidated = this.$MODERATIONcli.addModerationData(obj, itemStatus)
          return consolidated
        }))
        itemObject.data = newData
      }
      this.needsModerationData = false
      return itemObject
    },
    getIssues (resetPage) {
      this.isLoading = true
      const params = {
        q: this.query,
        page: resetPage ? 1 : this.pagination.page,
        page_size: this.pagination.pageSize,
        totalItems: undefined,
        sort: `${this.pagination.sortDesc ? '' : '-'}${this.pagination.sortBy}`
      }
      if (resetPage) { this.pagination.page = 1 }
      this.$APIcli._request(this.operationId, { params }).then(
        results => {
          this.issuesRequest = results.url
          this.issues = results.body
          this.needsModerationData = true
          this.pagination.totalItems = results.body.total
          this.isLoading = false
        },
        reason => console.error(`-C- IssuesList > failed on api call: ${reason}`)
      )
    },
    changeSelection (item) {
      if (this.isAuthenticated) {
        this.itemsSelection = this.$changeSelection(this.itemsSelection, item.id)
      }
    },
    isSelected (item) {
      return this.itemsSelection.includes(item.id)
    },
    callbackAction (evt) {
    },
    resetQuery () {
      this.query = undefined
      this.getIssues(true)
    },
    changePagination (pageNumber) {
      this.pagination.page = pageNumber
      this.getIssues()
    },
    changeSorting (sort) {
      this.pagination.sortBy = sort.sortBy
      this.pagination.sortDesc = sort.sortDesc
      this.getIssues()
    },
    formatDate (dateString, addTime) {
      return this.$formatDate(dateString, addTime)
    }
  }
}

</script>

<style>
  .table > tbody > tr > td {
    vertical-align: middle;
  }
  .table > tbody > tr > th {
    vertical-align: middle !important;
  }
  .table > thead > tr > th {
    vertical-align: middle !important;
  }
</style>
