<template>
  <div class="resource-card-component">

    <b-card
      header-tag="header"
      :header="cardTitle"
      footer-tag="footer"
      :footer="cardFooter"
      class="mt-3 mx-auto text-center"
      :style="`width: ${width};`"
      >

      <RawData
        :customClass="`mb-3`"
        :see="true"
        :dataRaw="resource"
      ></RawData>

      <!-- VIEW -->
      <div v-if="resource">
        <hr>
        <b-card-text>
          Resource title :<br>
          {{ resource.title }}
        </b-card-text>

        <!-- EDIT -->
        <b-button
          v-if="isAuthenticated && !edit"
          @click="edit=true"
          variant="primary"
          >
          <b-icon icon="pencil" aria-hidden="true"></b-icon>
          {{ $t('actions.edit') }}
        </b-button>
      </div>

      <!-- EDIT -->
      <b-container v-if="resource && isAuthenticated && edit">
        <hr>
        <b-form @submit="commentResource">

          <!-- COMMENT -->
          <b-form-group
            id="input-group-comment"
            label="Comment"
            label-for="resource-comment"
            description="your comment ..."
            >
            <b-form-input
              id="resource-comment"
              v-model="comment"
              placeholder="comment resource..."
            ></b-form-input>
          </b-form-group>

          <!-- CLOSE ISSUE -->
          <b-form-checkbox
            id="checkbox-close-resource"
            v-model="closeResource"
            name="checkbox-close-resource"
            >
            {{ $t('basics.resources', { list: $t('actions.close') }) }}
          </b-form-checkbox>

          <hr>

          <div v-if="!isLoading">
            <b-button @click="edit=false" class="mx-2" variant="danger">
              <b-icon icon="x" aria-hidden="true"></b-icon>
              {{ $t('actions.cancel') }}
            </b-button>
            <b-button type="submit" class="mx-2" variant="success">
              <b-icon icon="check2" aria-hidden="true"></b-icon>
              {{ $t('actions.comment') }}
            </b-button>
          </div>
          <div v-else>
            <b-spinner label="loading"></b-spinner>
          </div>

        </b-form>

      </b-container>

      <!-- EMPTY -->
      <div v-if="!resource">
        <!-- {{ defaultText }} -->
        <b-spinner label="loading"></b-spinner>
      </div>

    </b-card>
  </div>

</template>

<script>
import { mapState, mapGetters } from 'vuex'

import RawData from '@/components/ux/RawData.vue'

export default {
  name: 'ResourceCard',
  components: {
    RawData
  },
  props: [
    'cardTitle',
    'cardFooter',
    'resourceData',
    'resourceId',
    'width',
    'height'
  ],
  data () {
    return {
      edit: false,
      isLoading: false,
      defaultText: 'resource is loading',
      putOperationId: 'comment_resource',
      resource: undefined,
      comment: '',
      closeResource: false
    }
  },
  created () {
    console.log('-C- ResourceCard > created ... ')
  },
  watch: {
    resourceData (next) {
      if (next) {
        this.resource = next
      }
    }
  },
  computed: {
    ...mapState({
      log: (state) => state.global.log
    }),
    ...mapGetters({
      isAuthenticated: 'oauth/isAuthenticated'
    })
  },
  methods: {
    commentResource (evt) {
      evt.preventDefault()
      const API = this.$APIcli
      console.log('-C- ResourceCard > methods > commentResource > API :', API)
      this.isLoading = true
      const params = {
        id: this.resourceId,
        payload: {
          comment: this.comment,
          close: this.closeResource
        }
      }
      const body = {}
      API._request(this.putOperationId, { params, body, needAuth: true }).then(
        results => {
          this.isLoading = false
          console.log('-C- ResourceCard > methods > commentResource > results.body :', results.body)
          this.resource = results.body
        },
        reason => {
          console.error(`-C- ResourceCard > failed on api call: ${reason}`)
          this.isLoading = false
        }
      )
    }
  }
}

</script>