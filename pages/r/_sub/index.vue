<template>
  <v-container fluid pa-0 v-scroll="onScroll">
    <v-layout column>
      <v-flex mb-1>
        <v-jumbotron color="primary" style="height: auto">
          <v-container pa-0>
            <v-layout row wrap align-center justify-center>
              <v-flex xs12 sm11 md11 lg10 xl8 py-4>
                <v-card color="primary" class="white--text" flat>
                  <v-card-title primary-title><h1 class="display-2">r/{{subreddit}}</h1></v-card-title>
                  <v-card-text>
                    <p>
                      <b>{{numberWithCommas(subscribers)}}</b> subscribers
                      <br>
                      {{'created in ' + new Date(created_utc * 1000).getFullYear()}}
                    </p>
                    <!-- <div>
                      <b>{{numberWithCommas(commenters)}}</b> commenters in our dataset
                    </div> -->
                    <br>
                    <v-card>
                      <v-card-text>
                        <div class="reddit-html"  v-html="nofollowLink(public_description)"></div>
                        <a rel="noopener nofollow" target="_blank" :href="'https://reddit.com/r/' + subreddit">{{'https://reddit.com/r/' + subreddit}} <v-icon small color="primary">launch</v-icon></a>
                        <!-- <v-switch color="secondary" label="Show full description" v-model="full_description"></v-switch>
                        <div v-if="full_description" class="reddit-html" v-html="nofollowLink(description)"></div> -->
                      </v-card-text>
                    </v-card>
                  </v-card-text>
                </v-card>
              </v-flex>
            </v-layout>
          </v-container>
        </v-jumbotron>
      </v-flex>
      <v-flex mb-1>
        <v-container pa-2>
          <v-layout row wrap align-center justify-center>
            <v-flex xs12 sm11 md11 lg10 xl8>
              <v-radio-group v-model="sortSubreddits" label="sort by" row>
                <!-- <v-radio color="primary" label="This subreddit => Other subreddit" value="parent"></v-radio> -->
                <v-radio color="secondary" label="small specific" value="child"></v-radio>
                <v-radio color="primary" label="large general" value="combined"></v-radio>
                <!-- <v-radio color="accent" label="only products" value="ads"></v-radio> -->
              </v-radio-group>
            </v-flex>
          </v-layout>
        </v-container>
      </v-flex>
      <v-flex mb-5>
        <v-container pa-0>
          <v-layout row wrap align-center justify-center>
            <v-flex xs12 sm11 md11 lg10 xl8 mb-2 v-for="sub in x_subs_pretty_paginate" :key="x_subs_pretty.subreddits">
              <!-- Subreddit -->
              <v-card v-if="sub.subreddits">
                <v-card-title class="headline"><nuxt-link :to="'/r/' + sub.subreddits + '/'">r/{{sub.subreddits}}</nuxt-link><v-spacer></v-spacer><v-chip v-if="sub.over18" color="red" label outline>NSFW</v-chip><v-subheader>{{abbreviateNumber(sub.subscribers)}} subscribers</v-subheader></v-card-title>
                <v-card-text class="pt-0">
                  <v-card color="grey lighten-4" flat>
                    <v-card-text>
                      <div class="reddit-html" v-html="nofollowLink(sub.public_description)"></div>
                      <a rel="noopener nofollow" target="_blank" :href="'https://reddit.com/r/' + sub.subreddits">{{'https://reddit.com/r/' + sub.subreddits}} <v-icon small color="primary">launch</v-icon></a>
                    </v-card-text>
                  </v-card>

                  <!-- <div><b>{{numberWithCommas(sub.commenters)}}</b> users commented on <b>r/{{sub.subreddits}}</b></div>
                  <div><b>{{numberWithCommas(sub.cross_commenters)}}</b> users commented on both <b>r/{{subreddit}}</b> and <b>r/{{sub.subreddits}}</b></div>
                  <div><b>{{roundNumber(sub.percent_of_parent_sub, 2)}}%</b> of users who commented on <b>r/{{subreddit}}</b> also commented on <b>r/{{sub.subreddits}}</b></div>
                  <div><b>{{roundNumber(sub.percent_of_child_sub, 2)}}%</b> of users who commented on <b>r/{{sub.subreddits}}</b> also commented on <b>r/{{subreddit}}</b></div> -->

                  <div style="position: relative">
                    <v-progress-linear :height="30" :value="(100 / (sub.percent_of_parent_sub + sub.percent_of_child_sub)) * sub.percent_of_parent_sub" color="primary" background-color="secondary"></v-progress-linear>
                    <div style="position: absolute; top: 0; bottom: 0; left: 24px; color: white; z-index: 1; line-height: 30px">
                      <v-tooltip top>
                        <span slot="activator"><b>{{subreddit}}</b> {{roundNumber(sub.percent_of_parent_sub, 2)}}%</span>
                        <span><b>{{roundNumber(sub.percent_of_parent_sub, 2)}}%</b> of users who commented on <b>r/{{subreddit}}</b> also commented on <b>r/{{sub.subreddits}}</b></span>
                      </v-tooltip>
                    </div>
                    <div style="position: absolute; top: 0; bottom: 0; right: 24px; color: white; z-index: 1; line-height: 30px">
                      <v-tooltip top>
                        <span slot="activator">{{roundNumber(sub.percent_of_child_sub, 2)}}% <b>{{sub.subreddits}}</b></span>
                        <span><b>{{roundNumber(sub.percent_of_child_sub, 2)}}%</b> of users who commented on <b>r/{{sub.subreddits}}</b> also commented on <b>r/{{subreddit}}</b></span>
                      </v-tooltip>
                    </div>
                  </div>
                </v-card-text>
              </v-card>
            </v-flex>
          </v-layout>
        </v-container>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<style>
.text--soften {
  opacity: 0.5;
}
.quote--smaller {
  font-size: 14px;
}
</style>

<script>
  import api from '~/my_modules/api'

  function percent (small, big) {
    return (small / big) * 100
  }

  export default {
    head () {
      return {
        title: `Subreddits similar to r/${this.subreddit} - reddit.guide`,
        meta: [
          { hid: 'description', name: 'description', content: `Find more subreddits like r/${this.subreddit}` },
          { hid: 'og:description', property: 'og:description', content: `Find more subreddits like r/${this.subreddit}` },
          { hid: 'og:title', property: 'og:title', content: `r/${this.subreddit} - reddit.guide` },
          { hid: 'og:url', property: 'og:url', content: `https://reddit.guide/r/${this.subreddit}/` }
        ]
      }
    },
    data () {
      return {
        paginate: 20,
        paginate_by: 10,
        ad_every_x: 5,
        full_description: false
      }
    },
    methods: {
      nofollowLink (link) {
        return link.replace(/<a/ig, '<a rel="noopener nofollow" target="_blank" ')
      },
      killLink (link) {
        return link.replace(/<a/ig, '<span class="secondary--text"').replace(/a>/ig, 'span>')
      },
      paginateLoad () {
        this.paginate += this.paginate_by
      },
      onScroll (e) {
        let scrollTop = window.pageYOffset || document.documentElement.scrollTop
        let viewHeight = window.innerHeight || document.documentElement.clientHeight
        let pageHeight = document.documentElement.scrollHeight || document.documentElement.offsetHeight
        let scrollBottom = scrollTop + viewHeight

        if (scrollBottom === pageHeight) this.paginateLoad()
      },
      numberWithCommas (x) {
        return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',')
      },
      abbreviateNumber (value) {
        var newValue = value
        if (value >= 1000) {
          var suffixes = ['', 'k', 'm', 'b', 't']
          var suffixNum = Math.floor((('' + value).length - 1) / 3)
          var shortValue = '' + Math.round(value / Math.pow(1000, suffixNum))
          newValue = shortValue + suffixes[suffixNum]
        }
        return newValue
      },
      roundNumber (x, decimal) {
        return Math.floor(x * Math.pow(10, decimal)) / Math.pow(10, decimal)
      },
      percent (small, big) {
        return (small / big) * 100
      }
    },
    computed: {
      sortSubreddits: {
        get: function () {
          return this.$store.state.toggles.sortSubreddits
        },
        set: function (value) {
          this.$store.commit('toggles/setSortSubreddits', value)
        }
      },
      showProducts: {
        get: function () {
          return this.$store.state.toggles.showProducts
        },
        set: function (value) {
          this.$store.commit('toggles/setShowProducts', value)
        }
      },
      showComments: {
        get: function () {
          return this.$store.state.toggles.showComments
        },
        set: function (value) {
          this.$store.commit('toggles/setShowComments', value)
        }
      },
      x_subs_pretty () {
        let arr = []
        this.x_subs.subreddits.forEach((subreddit, index) => {
          let objectify = {}
          Object.keys(this.x_subs).forEach(key => {
            objectify[key] = this.x_subs[key][index]
          })

          objectify['percent_of_child_sub'] = percent(objectify['cross_commenters'], objectify['commenters'])
          objectify['percent_of_parent_sub'] = percent(objectify['cross_commenters'], this.commenters)
          arr.push(objectify)
        })

        // sort
        if (this.sortSubreddits === 'parent') {
          arr.sort((a, b) => {
            if (a.rank_parent === b.rank_parent) return 0
            return a.rank_parent < b.rank_parent ? -1 : 1
          })
        } else if (this.sortSubreddits === 'child') {
          arr.sort((a, b) => {
            if (a.rank_child === b.rank_child) return 0
            return a.rank_child < b.rank_child ? -1 : 1
          })
        } else { // combined
          arr.sort((a, b) => {
            if (a.rank_combined === b.rank_combined) return 0
            return a.rank_combined < b.rank_combined ? -1 : 1
          })
        }

        return arr
      },
      x_subs_pretty_paginate () {
        return this.x_subs_pretty.slice(0, this.paginate)
      }
    },
    validate ({ params }) {
      // Must alphanumeric
      return /^[A-Za-z0-9_\-.:]*$/.test(params.sub)
    },
    async asyncData ({params, error, payload}) {
      return api.getSubredditData(params.sub).then((data) => {
        return data
      }).catch((e) => {
        error({ statusCode: 404, message: 'Page not found' })
      })
    }
  }
</script>
