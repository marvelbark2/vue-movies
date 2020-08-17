<template>
  <main class="main">
    <TopNav
      :title="metaTitle" />

    <Hero
      :item="item" />

    <MediaNav
      :menu="menu"
      @clicked="navClicked" />

    <template v-if="activeMenu === 'overview'">
      <MovieInfo
        :item="item" />

      <Credits
        v-if="showCredits"
        :people="item.credits.cast" />
    </template>

    <template v-if="activeMenu === 'stream'">
      <Videos
        :videos="myvideo" />
    </template>

    <template v-if="activeMenu === 'photos' && showImages">
      <Images
        v-if="item.images.backdrops.length"
        title="Backdrops"
        type="backdrop"
        :images="item.images.backdrops" />

      <Images
        v-if="item.images.posters.length"
        title="Posters"
        type="poster"
        :images="item.images.posters" />
    </template>

    <ListingCarousel
      v-if="recommended && recommended.results.length"
      title="More Like This"
      :items="recommended" />
  </main>
</template>

<script>
import { apiImgUrl, getMovie, getMovieRecommended } from '~/api';
import { name, yearStart } from '~/mixins/Details';
import TopNav from '~/components/global/TopNav';
import Hero from '~/components/Hero';
import MediaNav from '~/components/MediaNav';
import MovieInfo from '~/components/movie/MovieInfo';
import Videos from '~/components/Videos';
import Images from '~/components/Images';
import Credits from '~/components/Credits';
import ListingCarousel from '~/components/ListingCarousel';
export default {
  components: {
    TopNav,
    Hero,
    MediaNav,
    MovieInfo,
    Videos,
    Images,
    Credits,
    ListingCarousel,
  },
  mixins: [
    name,
    yearStart,
  ],
  head () {
    return {
      title: this.metaTitle,
      meta: [
        { hid: 'og:title', property: 'og:title', content: this.metaTitle },
        { hid: 'og:description', property: 'og:description', content: this.metaDescription },
        { hid: 'description', name: 'description', content: this.metaDescription },
        { hid: 'og:image', property: 'og:image', content: this.metaImage },
        { hid: 'og:url', property: 'og:url', content: `${process.env.FRONTEND_URL}${this.$route.path}` },
      ],
      bodyAttrs: {
        class: 'topnav-active',
      },
    };
  },
  data () {
    return {
      menu: [],
      activeMenu: 'overview',
      recommended: null,
      myvideo: [{ id: 1, key: '339BE5B6F215AB211226368305839D804C102A59', site: 'SelfServer', name: 'Myvideo', src: 'https://www.youtube.com/embed/MsdVLe2xi9g?rel=0&showinfo=0&autoplay=1', url: 'https://historical-deciduous-antimatter.glitch.me/stream/339BE5B6F215AB211226368305839D804C102A59', thumb: 'https://image.tmdb.org/t/p/w370_and_h556_bestv2/5MSDwUcqnGodFTvtlLiLKK0XKS.jpg', duration: 200, type: 'stream' }],
    };
  },
  computed: {
    metaTitle () {
      if (this.yearStart) {
        return `${this.name} (${this.yearStart})`;
      } else {
        return `${this.name}`;
      }
    },
    metaDescription () {
      if (this.item.overview) {
        return this.truncate(this.item.overview, 200);
      } else {
        return '';
      }
    },
    metaImage () {
      if (this.item.poster_path) {
        return `${apiImgUrl}/w500${this.item.poster_path}`;
      } else {
        return '';
      }
    },
    showCredits () {
      const credits = this.item.credits;
      return credits && credits.cast && credits.cast.length;
    },
    showVideos () {
      const videos = this.item.videos;
      return videos && videos.results && videos.results.length;
    },
    showImages () {
      const images = this.item.images;
      return images && ((images.backdrops && images.backdrops.length) || (images.posters && images.posters.length));
    },
  },
  async asyncData ({ params, error }) {
    try {
      const item = await getMovie(params.id);
      if (item.adult) {
        error({ message: 'This movie is not available' });
      } else {
        return { item };
      }
    } catch {
      error({ message: 'Page not found' });
    }
  },
  created () {
    this.createMenu();
    this.initRecommended();
  },
  methods: {
    truncate (string, length) {
      return this.$options.filters.truncate(string, length);
    },
    createMenu () {
      const menu = [];
      // overview
      menu.push('Overview');
      // videos
      if (this.showVideos) menu.push('Videos');
      // images
      if (this.showImages) menu.push('Photos');
      // stream
      menu.push('Stream');
      this.menu = menu;
    },
    navClicked (label) {
      this.activeMenu = label;
    },
    initRecommended () {
      // if recommended don't exist, retreive them
      if (this.recommended !== null) return;
      getMovieRecommended(this.$route.params.id).then((response) => {
        this.recommended = response;
      });
    },
  },
};
</script>
