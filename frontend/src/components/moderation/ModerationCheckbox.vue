<template>
  <b-form
    inline
    class="justify-content-center">
    <b-form-checkbox
      v-model="item[field]"
      v-if="isAuthenticated"
      button
      button-variant="link"
      @change="updateModeration(item, field, $event)"
      >
      <b-icon
        :icon="getIcon(field)"
        :variant="getColor(field)"
        >
      </b-icon>
    </b-form-checkbox>
    <b-form-checkbox
      v-else
      disabled
      button
      variant="link"
      :value="item[field]"
      >
      <b-icon
        :icon="getIcon(field)"
        :variant="getColor(field)"
        >
      </b-icon>
    </b-form-checkbox>
  </b-form>
</template>

<script>

import { mapState, mapGetters } from 'vuex'

export default {
  name: 'ModerationCheckbox',
  props: [
    'dgfType',
    'item',
    'field'
  ],
  data () {
    return {
      buttons: {
        read: {
          icons: [
            'circle', 'check2-circle'
          ],
          colors: [
            'outline-secondary', 'success'
          ]
        },
        suspicious: {
          icons: [
            'triangle', 'exclamation-triangle-fill'
          ],
          colors: [
            'outline-secondary', 'warning'
          ]
        },
        deleted: {
          icons: [
            'octagon', 'x-octagon-fill'
          ],
          colors: [
            'outline-secondary', 'danger'
          ]
        },
        comments: {
          icons: [
            'chat-left', 'chat-left-text'
          ],
          colors: [
            'outline-info', 'info'
          ]
        }
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
    async updateModeration (item, field, evt) {
      const updatedItem = await this.$MODERATIONcli.updateModeration(this.dgfType, item, field, evt)
      console.log('-C- ModerationCheckbox > updateModeration > updatedItem : ', updatedItem)
    },
    getColor (field) {
      const fieldIcons = this.buttons[field]
      const bool = this.item[field]
      const index = Boolean(bool)
      const color = index ? fieldIcons.colors[1] : fieldIcons.colors[0]
      return color
    },
    getIcon (field) {
      const fieldIcons = this.buttons[field]
      const bool = this.item[field]
      const index = Boolean(bool)
      const icon = index ? fieldIcons.icons[1] : fieldIcons.icons[0]
      return icon
    }
  }
}
</script>
