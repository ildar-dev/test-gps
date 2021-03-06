<template lang="pug">
.map-page
  .markers-wrapper
    .title(v-if="markers.length") {{ $t('map.markersTitle') }}
    .no-markers(v-else) {{ $t('map.noMarkers') }}
    .markers
      marker-map(
        v-for="marker in markers"
        :key="marker.id"
        :id="marker.id"
        :coords="marker.coords"
        @click="onMarkerClick(marker)"
        :is-active="id === marker.id"
      )

  .map
    yandex-map(
      :coords="mapCoords"
      :markers="markers"
      @click="onMapClick"
      @marker-click="onMapMarkerClick"
    )
    .plus(@click="toggleAdding" :class="{ 'active': isAdding }")
      icon-plus
</template>
<script>
import IconPlus from '@/icons/plus';
import MarkerMap from '@/components/marker';
import YandexMap from '@/components/map';

import { mapState } from 'vuex';

export default {
  name: 'Map',
  components: {
    YandexMap,
    IconPlus,
    MarkerMap,
  },
  data: () => ({
    isAdding: false,
    mapCoords: [55.707298976442445, 37.625740654460806], // 2 Gamsonovsky per., Moscow
  }),
  props: {
    id: {
      type: [String, Number],
    },
  },
  computed: {
    ...mapState({
      markers: state => state.map.markers,
    }),
  },
  methods: {
    toggleAdding() {
      this.isAdding = !this.isAdding;
    },
    finishAdding() {
      this.isAdding = false;
    },
    async onMapClick(e) {
      this.selectMarker(undefined);
      if (!this.isAdding) {
        return;
      }
      this.$store.dispatch('addMarker', { id: this.markers.length + 1, coords: e.get('coords') });
      this.finishAdding();
    },
    onMarkerClick(marker) {
      this.mapCoords = marker.coords;
      this.selectMarker(marker.id);
    },
    selectMarker(id) {
      if (this.id === id) {
        return;
      }
      this.$router.replace({ params: { id } });
    },
    onMapMarkerClick(marker) {
      this.selectMarker(marker.id);
    },
  },
};
</script>
<style lang="stylus">
.map-page
  display flex
  .markers-wrapper
    width 25%
    .no-markers
      text-align center
      margin 32px 0
      color #d0e0e0
    .markers
      max-height calc(100vh - 175px)
      overflow auto
    .title
      font-size 24px
      margin 16px 0 24px 16px
    .marker
      cursor pointer
      padding 16px
      transition background-color .3s ease
      &.active
      &:hover
        background #e0e0e0
      .coords
        color #a0a0a0
        font-size 14px

  .map
    width 75%
    position relative
    .y-map
      width 100%
      height calc(100vh - 105px)
    .plus
      position absolute
      padding 24px
      border-radius 50%
      background #1776dc
      right 24px
      bottom 24px
      cursor pointer
      transition transform .3s ease
      &.active
        background red
        transform rotate(45deg)
      svg
        display block
        margin auto
        path
          fill #fff
  @media screen and (max-width: 640px)
    flex-direction column
    .markers-wrapper
      width 100%
      .markers
        max-height 165px
    .map
      width 100%
      .plus
        padding 12px 
        right 12px
        bottom 12px

</style>
