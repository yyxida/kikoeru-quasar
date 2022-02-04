<template>
  <div>
    <div class="text-h5 text-weight-regular q-ma-md">
      {{pageTitle}}
    </div>

    <div :class="`row justify-center q-mx-md`">
      <q-infinite-scroll @load="onLoad" :offset="250" :disable="stopLoad" style="max-width: 1680px;" class="col">

        <div class="row q-col-gutter-x-md q-col-gutter-y-lg" style="white-space: nowrap; flex-wrap: nowrap; overflow-x: scroll; max-width: 100%">
          <div class="col-xs-12 col-sm-6 col-md-4" :class="detailMode ? 'col-lg-3 col-xl-3': 'col-lg-2 col-xl-2'" v-for="recentwork in recentworks" :key="recentwork.id"  style="display: inline-block">
            <RecentCard :metadata="recentwork" :thumbnailMode="!detailMode" class="fit"/>
          </div>
        </div>

        <template v-slot:loading>
          <div class="row justify-center q-my-md">
            <q-spinner-dots color="primary" size="40px" />
          </div>
        </template>
      </q-infinite-scroll>
    </div>
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
      seed: 7, // random sort
      sortOption: {
        label: '按照发售日期新到老的顺序',
        order: 'release',
        sort: 'desc'
      },
      options: [
        {
          label: '按照发售日期新到老的顺序',
          order: 'release',
          sort: 'desc'
        },
        {
          label: '按照我的评价排序',
          order: 'rating',
          sort: 'desc'
        },
        {
          label: '按照发售日期老到新的顺序',
          order: 'release',
          sort: 'asc'
        },
        {
          label: '按照售出数量多到少的顺序',
          order: 'dl_count',
          sort: 'desc'
        },
        {
          label: '按照价格便宜到贵的顺序',
          order: 'price',
          sort: 'asc'
        },
        {
          label: '按照价格贵到便宜的顺序',
          order: 'price',
          sort: 'desc'
        },
        {
          label: '按照评价高到低的顺序',
          order: 'rate_average_2dp',
          sort: 'desc'
        },
        {
          label: '按照评论多到少的顺序',
          order: 'review_count',
          sort: 'desc'
        },
        {
          label: '按照RJ号大到小的顺序',
          order: 'id',
          sort: 'desc'
        },
        {
          label: '按照RJ号小到大的顺序',
          order: 'id',
          sort: 'asc'
        },
        {
          label: '按照全年龄新作优先的顺序',
          order: 'nsfw',
          sort: 'asc'
        },
        {
          label: '随机排序',
          order: 'random',
          sort: 'desc'
        }
      ]
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
</style>
