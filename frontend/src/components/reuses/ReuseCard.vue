<template>
  <div class="reuse-card-component">

    <b-card
      footer-tag="footer"
      :footer="cardFooter"
      >

      <template v-slot:header>
        <div class="d-flex flex-row justify-content-between align-items-center">
          <div class="flex-fill align-content-center">
            {{ cardTitle }}
          </div>
          <EditItemBtn
            :dgfType="dgfType"
            :endpoint="putOperationId"
            :item="reuse"
            :hideFields="['comment', 'spotlight', 'follow', 'share', 'contactProducer']"
            @responseAction="callbackAction"
            >
          </EditItemBtn>
        </div>
      </template>

      <!-- VIEW -->
      <div v-if="reuse">

        <CardTitle
          :title="reuse.title"
        />

        <CardProducer
          :item="reuse"
        />

        <CardDescription
          :text="reuse.description"
        />

      </div>

      <!-- EDIT -->
      <b-container v-if="reuse && isAuthenticated && edit">

        <b-form @submit="updateReuse">

          <!-- TITLE -->
          <b-form-group
            id="input-group-title"
            label="Title"
            label-for="reuse-title"
            description="the reuse's title..."
          >
            <b-form-input
              id="reuse-title"
              v-model="reuse.title"
              placeholder="Add title something..."
            ></b-form-input>
          </b-form-group>
          <hr>

          <!-- DESCRIPTION -->
          <b-form-group
            id="input-group-description"
            label="Description"
            label-for="textarea"
            description="the reuse's description..."
          >
            <b-form-textarea
              id="textarea"
              v-model="reuse.description"
              placeholder="Add a description ..."
              rows="3"
              max-rows="6"
            ></b-form-textarea>
          </b-form-group>
          <hr>

          <div v-if="!isLoading">
            <b-button @click="edit=false" class="mx-2" variant="danger">
              <b-icon icon="x" aria-hidden="true"></b-icon>
              {{ $t('actions.cancel') }}
            </b-button>
            <b-button type="submit" class="mx-2" variant="success">
              <b-icon icon="check2" aria-hidden="true"></b-icon>
              {{ $t('actions.save') }}
            </b-button>
          </div>
          <div v-else>
            <b-spinner label="loading"></b-spinner>
          </div>

        </b-form>

      </b-container>

      <!-- EMPTY -->
      <div v-if="!reuse">
        <!-- {{ defaultText }} -->
        <b-spinner label="loading"></b-spinner>
      </div>

      <RawData
        :customClass="`mb-3`"
        :see="seeRaw"
        :dataRaw="reuse"
      ></RawData>

    </b-card>
  </div>

</template>

<script>
import { mapState, mapGetters } from 'vuex'

import { APIoperations } from '@/config/APIoperations.js'

import CardTitle from '@/components/blocks/CardTitle.vue'
import CardProducer from '@/components/blocks/CardProducer.vue'
import CardDescription from '@/components/blocks/CardDescription.vue'

import EditItemBtn from '@/components/ux/EditItemBtn.vue'
import RawData from '@/components/ux/RawData.vue'

export default {
  name: 'ReuseCard',
  components: {
    CardTitle,
    CardProducer,
    CardDescription,
    EditItemBtn,
    RawData
  },
  props: [
    'cardTitle',
    'cardFooter',
    'reuseData',
    'reuseId',
    'height'
  ],
  data () {
    return {
      updateEndpoints: APIoperations.updateEndpoints,
      commentEndpoints: APIoperations.commentEndpoints,
      dgfType: 'reuse',
      edit: false,
      comment: false,
      seeRaw: true,
      isLoading: false,
      defaultText: 'reuse is loading',
      putOperationId: 'comment_reuse',
      reuse: undefined,
      commentContent: '',
      closeReuse: false
    }
  },
  watch: {
    reuseData (next) {
      if (next) {
        this.reuse = next
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
    callbackAction (evt) {
      console.log('-C- ReuseCard > callbackAction > evt : ', evt)
      switch (evt.category) {
        case 'openEdit':
          this.edit = true
          this.seeRaw = false
          break
        // case 'comment':
        //   this.comment = true
        //   this.seeRaw = false
        //   break
      }
    },
    updateReuse (evt) {
      evt.preventDefault()
      const API = this.$APIcli
      this.isLoading = true
      const params = {
        reuse: this.reuseId,
        payload: this.reuse
      }
      const body = {}
      API._request(this.putOperationId, { params, body, needAuth: true }).then(
        results => {
          this.isLoading = false
          this.reuse = results.body
        },
        reason => {
          console.error(`-C- ReuseCard > failed on api call: ${reason}`)
          this.isLoading = false
        }
      )
    }
  }
}

</script>
