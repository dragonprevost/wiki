<template lang='pug'>
  v-container(fluid, grid-list-lg)
    v-layout(row wrap)
      v-flex(xs12)
        .admin-header
          img(src='/svg/icon-paint-palette.svg', alt='Theme', style='width: 80px;')
          .admin-header-title
            .headline.primary--text Theme
            .subheading.grey--text Modify the look &amp; feel of your wiki
          v-spacer
          v-btn(color='success', depressed, @click='save', large, :loading='loading')
            v-icon(left) check
            span {{$t('common:actions.apply')}}
        v-form.pt-3
          v-layout(row wrap)
            v-flex(lg6 xs12)
              v-card.wiki-form
                v-toolbar(color='primary', dark, dense, flat)
                  v-toolbar-title
                    .subheading Theme
                v-card-text
                  v-select(
                    :items='themes'
                    outline
                    background-color='grey lighten-2'
                    prepend-icon='palette'
                    v-model='selectedTheme'
                    label='Site Theme'
                    persistent-hint
                    hint='Themes affect how content pages are displayed. Other site sections (such as the editor or admin area) are not affected.'
                    )
                    template(slot='item', slot-scope='data')
                      v-list-tile-avatar
                        v-icon.blue--text(dark) filter_frames
                      v-list-tile-content
                        v-list-tile-title(v-html='data.item.text')
                        v-list-tile-sub-title(v-html='data.item.author')
                  v-divider.mt-3
                  v-switch(
                    v-model='darkMode'
                    label='Dark Mode'
                    color='primary'
                    persistent-hint
                    hint='Not recommended for accessibility. May not be supported by all themes.'
                    )
              v-card.wiki-form.mt-3
                v-toolbar(color='primary', dark, dense, flat)
                  v-toolbar-title
                    .subheading Code Injection
                  v-spacer
                  v-chip(label, color='white', small).primary--text coming soon
                v-card-text
                  v-textarea(
                    v-model='injectCSS'
                    label='CSS Override'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    hint='CSS code to inject after system default CSS'
                    auto-grow
                    disabled
                    )
                  v-textarea.mt-2(
                    v-model='injectHeader'
                    label='Head HTML Injection'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    hint='HTML code to be injected just before the closing head tag'
                    auto-grow
                    disabled
                    )
                  v-textarea.mt-2(
                    v-model='injectFooter'
                    label='Body HTML Injection'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    hint='HTML code to be injected just before the closing body tag'
                    auto-grow
                    disabled
                    )
            v-flex(lg6 xs12)
              v-card
                v-toolbar(color='teal', dark, dense, flat)
                  v-toolbar-title
                    .subheading Download Themes
                  v-spacer
                  v-chip(label, color='white', small).teal--text coming soon
                v-card-text.caption -- Coming soon --
</template>

<script>
import _ from 'lodash'
import { sync } from 'vuex-pathify'

import themeSaveMutation from 'gql/admin/theme/theme-mutation-save.gql'

export default {
  data() {
    return {
      loading: false,
      themes: [
        { text: 'Default', author: 'requarks.io', value: 'default' }
      ],
      selectedTheme: 'default',
      darkModeInitial: false,
      injectCSS: '',
      injectHeader: '',
      injectFooter: ''
    }
  },
  computed: {
    darkMode: sync('site/dark')
  },
  mounted() {
    this.darkModeInitial = this.darkMode
  },
  beforeDestroy() {
    this.darkMode = this.darkModeInitial
  },
  methods: {
    async save () {
      this.loading = true
      this.$store.commit(`loadingStart`, 'admin-theme-save')
      try {
        const respRaw = await this.$apollo.mutate({
          mutation: themeSaveMutation,
          variables: {
            theme: this.selectedTheme,
            darkMode: this.darkMode
          }
        })
        const resp = _.get(respRaw, 'data.theming.setConfig.responseResult', {})
        if (resp.succeeded) {
          this.darkModeInitial = this.darkMode
          this.$store.commit('showNotification', {
            message: 'Theme settings updated successfully.',
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-theme-save')
      this.loading = false
    }
  }
}
</script>

<style lang='scss'>

</style>
