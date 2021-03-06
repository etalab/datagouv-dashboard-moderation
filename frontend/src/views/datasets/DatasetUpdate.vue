<template>
  <div class="dataset_update">

    <b-breadcrumb
      class="mb-5"
      :items="crumbs">
    </b-breadcrumb>

    <PageHeader
      :dgfType="'dataset'"
      :customClass="'mb-4'"
      >
      <template v-slot:badge>
        <div>
          {{ $t('navigation.from') }} :
          <span v-if="datasetsRequest">
            <a :href="datasetsRequest" target="_blank">
              JSON
            </a>
            |
            <a :href="dataset.page" target="_blank">
              datagouv public page
            </a>
          </span>
          <span v-else>
            {{ getOperationId }}
          </span>
        </div>
      </template>
    </PageHeader>

    <b-row class="mx-2">

      <!-- DISPLAY DATASET -->
      <b-col>
        <DatasetCard
          :cardFooter="undefined"
          :datasetData="dataset"
          :datasetId="datasetId"
          height="800px"
          >
        </DatasetCard>
      </b-col>

      <!-- MODERATION BOX -->
      <b-col sm="6" md="4">
        <ModerationRowCard
          :hasHeader="true"
          :dgfType="dgfType"
          :endpoint="endpointModeration"
          :item="dataset"
        />
      </b-col>

    </b-row>

  </div>
</template>

<script>
import { mapState } from 'vuex'

import PageHeader from '@/components/ux/PageHeader.vue'
import ModerationRowCard from '@/components/moderation/ModerationRowCard.vue'

import DatasetCard from '@/components/datasets/DatasetCard.vue'

export default {
  name: 'DatasetUpdate',
  components: {
    PageHeader,
    ModerationRowCard,
    DatasetCard
  },
  data () {
    return {
      isLoading: false,
      dgfType: 'dataset',
      getOperationId: 'get_dataset',
      putOperationId: 'update_dataset',
      endpointModeration: 'dataset',
      datasetId: this.$route.params.id,
      datasetsRequest: undefined,
      dataset: undefined,
      needsModerationData: false,
      crumbs: [
        {
          text: this.$t('home.name'),
          to: '/'
        },
        {
          text: this.$t('basics.datasets', { list: '' }),
          to: '/datasets'
        },
        {
          text: '...', // this.$route.params.id,
          active: true
        }
      ]
    }
  },
  created () {
    this.getDataset()
  },
  watch: {
    async dataset (next) {
      if (next && this.needsModerationData) {
        this.dataset = await this.appendModerationData(next)
      }
    }
  },
  computed: {
    ...mapState({
      log: (state) => state.log
    })
  },
  methods: {
    async appendModerationData (itemObject) {
      const itemStatus = await this.$MODERATIONcli.getModeration(itemObject.id)
      console.log('-V- DatasetUpdate > methods > getDataset > itemStatus :', itemStatus)
      const consolidated = this.$MODERATIONcli.addModerationData(itemObject, itemStatus)
      console.log('-V- DatasetUpdate > methods > getDataset > consolidated :', consolidated)
      this.needsModerationData = false
      return consolidated
    },
    async getDataset () {
      const API = this.$APIcli
      const params = { dataset: this.datasetId }
      this.isLoading = true
      API._request(this.getOperationId, { params }).then(
        results => {
          this.datasetsRequest = results.url
          this.dataset = results.body
          const title = this.dataset.title.length > 25 ? this.dataset.title.slice(0, 25) + '...' : this.dataset.title
          this.crumbs[2].text = title
          this.isLoading = false
          this.needsModerationData = true
        },
        reason => {
          console.error(`failed on api call: ${reason}`)
          this.isLoading = false
        }
      )
    }
  }
}

</script>
