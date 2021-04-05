<template>
  <div class="page-wrapper">
    <template v-if="$fetchState.pending && !articles.length">
      <div class="article-card-wrapper">
        <content-placeholders
          v-for="p in 30"
          :key="p"
          rounded
          class="article-card-block"
        >
          <content-placeholder-img />
          <content-placeholder-text :lines="3" />
        </content-placeholders>
      </div>
    </template>
    <template v-else-if="$fetchState.error">
      <inline-error-block :error="$fetchState.error" />
    </template>
    <template v-else>
      <div class="article-cards-wrapper">
        <article-card-block
          v-for="(article, i) in articles"
          v-observe-visibility="
            i === articles.length - 1 ? lazyLoadArticles : false
          "
          :key="article.id"
          :article="article"
          class="article-card-block"
        ></article-card-block>
      </div>
    </template>
  </div>
</template>

<script>
import ArticleCardBlock from "@/components/blocks/ArticleCardBlock";
import InlineErrorBlock from "@/components/blocks/InlineErrorBlock";

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export default {
  components: {
    ArticleCardBlock,
    InlineErrorBlock,
  },
  data() {
    return {
      currentPage: 1,
      articles: [],
    };
  },
  async fetch() {
    const articles = await fetch(
      `https://dev.to/api/articles?tag=${this.$route.params.tag}&top=365&page=${this.currentPage}`
    ).then((res) => res.json());

    this.articles = this.articles.concat(articles);
  },
  methods: {
    lazyLoadArticles(isVisible) {
      if (isVisible) {
        if (this.currentPage < 5) {
          this.currentPage++;
          this.$fetch();
        }
      }
    },
  },
  head() {
    return {
      title:
        this.$route.params.tag &&
        `${capitalize(this.$route.params.tag)} articles`,
    };
  },
};
</script>

<style lang="scss" scoped>
.page-wrapper {
  max-width: $screen-xl;
  margin: auto;
  padding: 1rem;
  min-height: 100vh;
}

.article-cards-wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  .article-card-block {
    width: calc(100% - 2 * 1rem);
    margin: 1rem;
    margin-bottom: 1.5rem;
    margin-top: 0.5rem;
    @media (min-width: $screen-sm) {
      width: calc(50% - 2 * 1rem);
    }
    @media (min-width: $screen-lg) {
      width: calc(33.33333% - 2 * 1rem);
    }
  }
}
</style>
