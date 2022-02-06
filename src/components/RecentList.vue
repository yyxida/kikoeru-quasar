<template>
  <div>
    <div class="text-h5 text-weight-regular q-ma-md">
      {{pageTitle}}
    </div>

    <div :class="`row justify-center q-mx-md`">
      <q-infinite-scroll
      @load="onLoad"
      :offset="250"
      :disable="stopLoad"
      style="max-width: 1680px; overflow-x: scroll; padding: 15px;"
      class="col">
        <div class="row q-col-gutter-x-md q-col-gutter-y-lg no-wrap">
          <div class="col-md-2 col-xs-6 col-sm-4" v-for="recentwork in recentworks" :key="recentwork.id"  style="display: inline-block">
            <RecentCard :metadata="recentwork" :thumbnailMode="!detailMode" class="fit"/>
          </div>
        </div>
      </q-infinite-scroll>
    </div>

    <!-- <div class="btn-right">
      <span style="display: inline; padding: 10px"> > </span>
    </div> -->
  </div>
</template>

<script>
import RecentCard from 'components/RecentCard'
import NotifyMixin from '../mixins/Notification.js'

export default {
  name: 'RecentList',

  mixins: [NotifyMixin],

  components: {
    RecentCard
  },

  data () {
    return {
      listMode: false,
      showLabel: true,
      detailMode: true,
      stopLoad: false,
      recentworks: [],
      pageTitle: '',
    }
  },

  created () {
    this.refreshPageTitle();
    this.seed = Math.floor(Math.random() * 100);
  },

  mounted() {
    if (localStorage.sortOption) {
      try {
        this.sortOption = JSON.parse(localStorage.sortOption);
      } catch {
        localStorage.removeItem('sortOption');
      }
    }
    if (localStorage.showLabel) {
      this.showLabel = (localStorage.showLabel === 'true');
    }
    if (localStorage.listMode) {
      this.listMode = (localStorage.listMode === 'true');
    }
    if (localStorage.detailMode) {
      this.detailMode = (localStorage.detailMode === 'true');
    }
  },

  computed: {
    url () {
      const query = this.$route.query
      if (query.circleId) {
        return `/api/circles/${this.$route.query.circleId}/works`
      } else if (query.tagId) {
        return `/api/tags/${this.$route.query.tagId}/works`
      } else if (query.vaId) {
        return `/api/vas/${this.$route.query.vaId}/works`
      } else if (query.keyword) {
        return `/api/search/${query.keyword}`
      } else {
        return '/api/works'
      }
    }
  },

  // keep-alive hooks
  // <keep-alive /> is set in MainLayout
  activated () {
    this.stopLoad = false
    this.requestRecentWorks()
  },

  deactivated () {
    this.stopLoad = true
  },

  watch: {
    url () {
      this.reset()
    },

    sortOption (newSortOptionSetting) {
      localStorage.sortOption = JSON.stringify(newSortOptionSetting);
      this.seed = Math.floor(Math.random() * 100);
      this.reset();
    },

    showLabel (newLabelSetting) {
      localStorage.showLabel = newLabelSetting;
    },

    listMode (newListModeSetting) {
      localStorage.listMode = newListModeSetting;
    },

    detailMode(newModeSetting) {
      localStorage.detailMode = newModeSetting;
    },
  },

  methods: {
    onLoad () {
      this.requestRecentWorks()
    },

    requestRecentWorks () {
        this.$axios.get('/api/history/recent')
        .then((response) => {
            console.log(response)

            this.recentworks = response.data

            this.stopLoad = true
        })
        .catch((error) => {
            console.log(error)

            this.stopLoad = true
        })
    },

    refreshPageTitle () {
        this.pageTitle = '最近播放'
    },

    reset () {
      this.stopLoad = true
      this.refreshPageTitle()
      this.pagination = { currentPage:0, pageSize:12, totalCount:0 }
      this.requestWorksQueue()
        .then(() => {
          this.stopLoad = false
        })
    },
  }
}
</script>

<style lang="scss" scoped>
  .list {
    // 宽度 >= $breakpoint-sm-min
    @media (min-width: $breakpoint-sm-min) {
      padding: 0px 20px;
    }
  }

  .work-card {
    // 宽度 > $breakpoint-xl-min
    @media (min-width: $breakpoint-md-min) {
      width: 560px;
    }
  }

  .btn-right {
    float: right;
    font-size:32px;
    position: relative;
    top: -75px;
    background: rgba(71, 71, 71, 0.548);
    cursor: hand;
    z-index: 1000;
  }
</style>
