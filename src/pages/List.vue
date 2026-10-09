<template>
  <q-page v-if="postList.length !== 0" padding>
    <q-list padding class="rounded-borders" style="margin-top: -24px;">
      <Item :postList="postList" />
    </q-list>
    <div class="row justify-center q-mt-xl q-mb-lg" v-if="totalPages > 1">
      <q-pagination
        v-model="currentPage"
        :max="totalPages"
        :max-pages="7"
        direction-links
        boundary-numbers
        ellipses
        @input="onPageChange"
      >
        <template v-slot:prev>
          <span class="q-px-sm">Previous</span>
        </template>
        <template v-slot:next>
          <span class="q-px-sm">Next</span>
        </template>
      </q-pagination>
    </div>
  </q-page>
</template>


<script>
import { axiosInstance } from 'boot/axios';
import Item from '../components/Item';

const PER_PAGE = 5;

export default {
  name: 'List',
  components: { Item },
  data() {
    return {
      postList: [],
      currentPage: 1,
      totalPages: 1,
    };
  },
  watch: {
    '$route'() {
      this.getIssueList();
    },
  },
  methods: {
    getIssueList() {
      this.$q.loading.show({ delay: 250 });
      const page = parseInt(this.$route.query.page, 10) || 1;
      this.currentPage = page;
      let url = `/search/issues?q=+repo:${this.$store.getters.repositorySlug}+state:open+is:issue&page=${page}&per_page=${PER_PAGE}`;
      if (this.$route.query.label) {
        url += `+label:${this.$route.query.label}`;
      }
      axiosInstance.get(url)
        .then((res) => {
          this.$set(this, 'postList', res.data.items);
          this.totalPages = Math.max(1, Math.ceil(res.data.total_count / PER_PAGE));
          this.$q.loading.hide();
        })
        .catch(() => {
          this.$q.loading.hide();
        });
    },
    onPageChange(page) {
      const query = { ...this.$route.query, page };
      if (page === 1) {
        delete query.page;
      }
      this.$router.push({ path: '/', query });
    },
  },
  created() {
    this.getIssueList();
  },
};
</script>


<style>
</style>
