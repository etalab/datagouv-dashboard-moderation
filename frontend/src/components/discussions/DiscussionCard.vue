<template>
  <div class="discussion-card-component">

    <b-card
      header-tag="header"
      :header="cardTitle"
      footer-tag="footer"
      :footer="cardFooter"
      class="mt-3 mx-auto text-center"
      :style="`width: ${width};`"
      >

      <!-- VIEW -->
      <div v-if="discussion">
        <hr>
        <b-card-text>
          Discussion title :<br>
          {{ discussion.title }}
        </b-card-text>
        <hr>
        <b-card-text>
          <code>
            {{ discussion }}
          </code>
        </b-card-text>

        <!-- EDIT -->
        <b-button
          v-if="isAuthenticated && !edit"
          @click="edit=true"
          variant="primary"
          >
          <b-icon icon="pencil" aria-hidden="true"></b-icon>
          edit
        </b-button>
      </div>

      <!-- EDIT -->
      <b-container v-if="discussion && isAuthenticated && edit">
        <hr>
        <b-form @submit="commentDiscussion">

          <!-- COMMENT -->
          <b-form-group
            id="input-group-comment"
            label="Comment"
            label-for="discussion-comment"
            description="your comment ..."
            >
            <b-form-input
              id="discussion-comment"
              v-model="comment"
              placeholder="comment discussion..."
            ></b-form-input>
          </b-form-group>

          <!-- CLOSE ISSUE -->
          <b-form-checkbox
            id="checkbox-close-discussion"
            v-model="closeDiscussion"
            name="checkbox-close-discussion"
            >
            Close discussion
          </b-form-checkbox>

          <hr>

          <div v-if="!isLoading">
            <b-button type="submit" variant="primary">
              <b-icon icon="check2" aria-hidden="true"></b-icon>
              comment
            </b-button>
            &nbsp;
            <b-button @click="edit=false" variant="light">
              <b-icon icon="x" aria-hidden="true"></b-icon>
              cancel
            </b-button>
          </div>
          <div v-else>
            <b-spinner label="loading"></b-spinner>
          </div>

        </b-form>

      </b-container>

      <!-- EMPTY -->
      <div v-if="!discussion">
        <!-- {{ defaultText }} -->
        <b-spinner label="loading"></b-spinner>
      </div>

    </b-card>
  </div>

</template>

<script>
import { mapState, mapGetters } from 'vuex'

export default {
  name: 'DiscussionCard',
  props: [
    'cardTitle',
    'cardFooter',
    'discussionData',
    'discussionId',
    'width',
    'height'
  ],
  data () {
    return {
      edit: false,
      isLoading: false,
      defaultText: 'discussion is loading',
      putOperationId: 'comment_discussion',
      discussion: undefined,
      comment: '',
      closeDiscussion: false
    }
  },
  created () {
    console.log('-C- DiscussionCard > created ... ')
  },
  watch: {
    discussionData (next) {
      if (next) {
        this.discussion = next
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
    commentDiscussion (evt) {
      evt.preventDefault()
      const API = this.$APIcli
      console.log('-C- DiscussionCard > methods > commentDiscussion > API :', API)
      this.isLoading = true
      const params = {
        id: this.discussionId,
        payload: {
          comment: this.comment,
          close: this.closeDiscussion
        }
      }
      const body = {}
      API._request(this.putOperationId, { params, body, needAuth: true }).then(
        results => {
          this.isLoading = false
          console.log('-C- DiscussionCard > methods > commentDiscussion > results.body :', results.body)
          this.discussion = results.body
        },
        reason => {
          console.error(`-C- DiscussionCard > failed on api call: ${reason}`)
          this.isLoading = false
        }
      )
    }
  }
}

</script>