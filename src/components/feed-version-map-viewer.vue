<template>
  <div style="position:relative">
    <b-message v-if="error" class="is-danger">
      {{ error }}
    </b-message>
    <div v-else-if="$apollo.loading" class="is-loading">
      Loading
    </div>
    <div v-else>
      <tl-map-viewer
        :enable-scroll-zoom="enableScrollZoom"
        :features="features"
        :route-features="routeFeatures"
        :stop-features="stopFeatures"
        :center="center"
        :auto-fit="true"
        :zoom="zoom ? zoom : null"
        @setAgencyFeatures="agencyFeatures = $event"
        @mapClick="mapClick"
        @setZoom="currentZoom = $event"
      />
      <div v-if="overlay" class="map-panel">
        <tl-map-route-list
          :current-zoom="currentZoom"
          :link-version="linkVersion"
          :agency-features="agencyFeatures"
          :is-component-modal-active="isComponentModalActive"
          @close="isComponentModalActive = false"
        />
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const q = gql`
query ($limit: Int!, $agency_ids: [Int!], $route_ids: [Int!], $feed_version_sha1: String, $include_stops: Boolean! = false) {
  routes(limit: $limit, ids: $route_ids, where: {agency_ids: $agency_ids, feed_version_sha1: $feed_version_sha1}) {
    id
    onestop_id
    feed_onestop_id
    feed_version_sha1
    route_id
    route_color
    route_desc
    route_long_name
    route_short_name
    route_type
    route_url
    geometries {
      geometry
      combined_geometry
      generated
      length
      max_segment_length
    }
    headways {
      dow_category
      direction_id
      headway_secs
    }
    route_stops @include(if: $include_stops) {
      stop {
        id
        stop_id
        stop_name
        geometry
      }
    }
    agency {
      id
      agency_id
      agency_name
    }
  }
}
`

export default {
  apollo: {
    routes: {
      client: 'transitland',
      query: q,
      error (e) { this.error = e },
      variables () {
        return {
          include_stops: this.includeStops,
          feed_version_sha1: this.feedVersionSha1,
          route_ids: this.routeIds,
          agency_ids: this.agencyIds,
          limit: 10000
        }
      }
    }
  },
  props: {
    feedVersionSha1: { type: String, default: null },
    includeStops: { type: Boolean, default: false },
    overlay: { type: Boolean, default: false },
    fvids: { type: Array, default: null },
    routeIds: { type: Array, default: null },
    agencyIds: { type: Array, default: null },
    linkVersion: { type: Boolean, default: false },
    features: { type: Array, default () { return [] } },
    center: { type: Array, default () { return [] } },
    zoom: { type: Number, default: null },
    enableScrollZoom: { type: Boolean, default: false }
  },
  data () {
    return {
      routes: [],
      error: null,
      isComponentModalActive: false,
      agencyFeatures: {},
      currentZoom: 0
    }
  },
  computed: {
    routeFeatures () {
      const features = []
      for (const feature of this.routes) {
        if (!feature.geometries || feature.geometries.length === 0) {
          continue
        }
        const geom = feature.geometries[0]
        let routeColor = feature.route_color
        if (routeColor && routeColor.substr(0, 1) !== '#') {
          routeColor = '#' + routeColor
        }
        const headwaySorted = (feature.headways || [])
          .filter((s) => { return s.dow_category === 1 })
          .sort((a, b) => { return a.direction_id < b.direction_id })
        const fcopy = Object.assign({}, feature, {
          geometry_length: geom.length || -1,
          generated: geom.generated,
          max_segment_length: geom.max_segment_length,
          route_color: routeColor,
          headway_secs: (headwaySorted.length > 0 ? headwaySorted[0].headway_secs : null) || -1,
          agency_name: feature.agency ? feature.agency.agency_name : null
        })
        delete fcopy.geometry
        delete fcopy.__typename
        features.push({
          id: feature.id,
          type: 'Feature',
          properties: fcopy,
          geometry: geom.combined_geometry
        })
      }
      return features
    },
    stopFeatures () {
      const features = []
      for (const feature of this.routes) {
        for (const g of feature.route_stops || []) {
          const fcopy = Object.assign({}, g.stop)
          delete fcopy.geometry
          delete fcopy.__typename
          features.push({
            type: 'Feature',
            geometry: g.stop.geometry,
            properties: fcopy,
            id: g.stop.id
          })
        }
      }
      return features
    }
  },
  watch: {
    routeFeatures (v) {
      this.$emit('setRouteFeatures', v)
    },
    stopFeatures (v) {
      this.$emit('setSopFeatures', v)
    }
  },
  methods: {
    mapClick (e) {
      if (Object.keys(this.agencyFeatures).length > 0) {
        this.isComponentModalActive = true
      }
    }
  }
}
</script>
