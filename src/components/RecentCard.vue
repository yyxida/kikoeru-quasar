<template>
  <q-card>
    <router-link :to="`/work/${metadata.work_id}`">
      <CoverSFW :workid="metadata.work_id" :nsfw="false" :release="true" style="max-width:200px;"/>
    </router-link>

    <q-separator />

    <div v-if="!thumbnailMode">
      <!-- 标题 -->
      <div class="q-mx-sm text-h6 text-weight-regular ellipsis-2-lines">
        <router-link :to="`/work/${metadata.work_id}`" class="text-black">
          {{ metadata.title }}
        </router-link>
      </div>
    </div>

    <div>
      <p>{{metadata.file_name}}</p>
    </div>
  </q-card>
</template>

<script>
// import WorkDetails from 'components/WorkDetails'
import CoverSFW from 'components/CoverSFW'
import NotifyMixin from '../mixins/Notification.js'

export default {
  name: 'RecentCard',

  mixins: [NotifyMixin],

  components: {
    CoverSFW
  },

  props: {
    metadata: {
      type: Object,
      required: true
    },
    thumbnailMode: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      rating: 0,
      userMarked: false,
      showTags: true
    }
  },

  computed: {
    sortedRatings: function() {
      function compare(a, b) {
        return (a.review_point > b.review_point) ? -1 : 1;
      }

      return this.metadata.rate_count_detail.slice().sort(compare);
    }
  },

  // TODO: Refactor with Vuex?
  mounted() {
    if (this.metadata.userRating) {
      this.userMarked = true;
      this.rating = this.metadata.userRating;
    } else {
      this.userMarked = false;
      this.rating = this.metadata.rate_average_2dp || 0;
    }

    // 极个别作品没有标签
    if (this.metadata.tags && this.metadata.tags[0].name === null) {
      this.showTags = false;
    }
  },

  watch: {
    rating (newRating, oldRating) {
      if (oldRating) {
        const submitPayload = {
          'user_name': this.$store.state.User.name, // 用户名不会被后端使用
          'work_id': this.metadata.id,
          'rating': newRating
        };
        this.userMarked = true;
        this.submitRating(submitPayload);
      }
    }
  },

  methods: {
    onClickPlay () {

    }
  }
}
</script>
