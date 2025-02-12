<template>
  <div>
    <div v-if="$apollo.loading" class="is-loading" />
    <div v-else-if="entity">
      <nav class="breadcrumb">
        <ul>
          <li>
            <nuxt-link :to="{name:'operators'}">
              Operators
            </nuxt-link>
          </li><li>
            <nuxt-link :to="{name:'operators-onestop_id', params:{onestop_id:$route.params.onestop_id}}">
              {{ operatorName }}
            </nuxt-link>
          </li>
        </ul>
      </nav>

      <h1 class="title">
        {{ operatorName }}
      </h1>

      <slot name="description">
        <div class="content">
          {{ staticDescription }}
        </div>
      </slot>

      <!-- Warnings for freshness and viewing a specific version -->
      <b-message v-if="dataFreshness > 365" type="is-warning" has-icon>
        The GTFS feeds associated with this page were fetched {{ dataFreshness }} days ago; use caution or check if newer data is available.
      </b-message>
      <b-message v-if="linkVersion" type="is-warning" has-icon>
        You are viewing a single GTFS Agency entity defined in source feed
        <nuxt-link :to="{name:'feeds-feed', params:{feed:$route.query.feed_onestop_id}}">
          {{ $route.query.feed_onestop_id | shortenName }}
        </nuxt-link> version
        <nuxt-link :to="{name:'feeds-feed-versions-version', params:{feed:$route.query.feed_onestop_id, version:$route.query.feed_version_sha1}}">
          {{ $route.query.feed_version_sha1 | shortenName(8) }}
        </nuxt-link>.<br>
        <template v-if="!search">
          Click <nuxt-link :to="{name: 'operators-onestop_id', params:{onestop_id:$route.params.onestop_id}}">
            here
          </nuxt-link> to return to the main view.
        </template>
      </b-message>

      <!-- Main content -->
      <div class="columns">
        <div class="column is-three-quarters">
          <table class="table is-borderless property-list">
            <tr>
              <td>
                <b-tooltip dashed label="A globally unique identifier for this operator">
                  Onestop ID
                </b-tooltip>
              </td>
              <td><code>{{ searchKey.onestop_id }}</code></td>
            </tr>
            <tr>
              <td>
                <b-tooltip dashed multiline label="Matched agencies; see 'Sources' below for full details">
                  Agencies
                </b-tooltip>
              </td>
              <td>
                <ul>
                  <li v-for="k of agencyNames" :key="k">
                    {{ k }}
                  </li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <b-tooltip dashed multiline label="Operators and their service areas are matched against place names from the Natural Earth project">
                  Locations
                </b-tooltip>
              </td>
              <td>
                <ul>
                  <li v-for="location of locations" :key="location.name">
                    <nuxt-link :to="{name:'operators', query:{adm0_name:location.adm0_name}}">
                      {{ location.adm0_name }}
                    </nuxt-link>
                    <template v-if="location.adm1_name">
                      /
                      <nuxt-link :to="{name:'operators', query:{adm1_name:location.adm1_name, adm0_name:location.adm0_name}}">
                        {{ location.adm1_name }}
                      </nuxt-link>
                    </template>
                    <template v-if="location.city_name">
                      /
                      <nuxt-link :to="{name:'operators', query:{city_name:location.city_name, adm1_name:location.adm1_name, adm0_name:location.adm0_name}}">
                        {{ location.city_name }}
                      </nuxt-link>
                    </template>
                  </li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                Contact
              </td>
              <td>
                <ul>
                  <li v-for="k of agencyURLs" :key="k">
                    <code>{{ k }}</code>
                  </li>
                </ul>
              </td>
            </tr>
            <tr v-if="entity && entity.tags && Object.keys(entity.tags).length > 0">
              <td>
                <b-tooltip dashed multiline label="Links between Transitland and other catalogs and data sources on the Internet">
                  ID Crosswalk
                </b-tooltip>
              </td>
              <td>
                <ul>
                  <li v-if="entity.tags.us_ntd_id">
                    US National Transit Database (NTD) ID: <code>{{ entity.tags.us_ntd_id }}</code> <a target="_blank" href="https://www.transit.dot.gov/ntd/"><b-icon icon="link" title="US National Transit Database website" /></a>
                  </li>
                  <li v-if="entity.tags.omd_provider_id">
                    OpenMobilityData Provider ID: <code>{{ entity.tags.omd_provider_id }}</code> <a target="_blank" :href="`https://openmobilitydata.org/p/${entity.tags.omd_provider_id}`"><b-icon icon="link" title="OpenMobilityData provider page" /></a>
                  </li>
                  <li v-if="entity.tags.wikidata_id">
                    Wikidata Entity ID: <code>{{ entity.tags.wikidata_id }}</code> <a target="_blank" :href="`https://www.wikidata.org/wiki/${entity.tags.wikidata_id}`"><b-icon icon="link" title="Wikidata entity query page" /></a>
                  </li>
                </ul>
              </td>
            </tr>
          </table>
        </div>

        <div class="column is-one-quarter is-full-height">
          <slot name="edit-operator" :entity="entity" />
        </div>
      </div>

      <hr>
      <h4 class="title is-4">
        Source Feed(s)
      </h4>

      <b-tabs type="is-boxed" :animated="false">
        <b-tab-item label="Source Feeds">
          <b-message
            v-for="feedSpec, feedOnestopId in uniqueFeedSourcesOnestopIds"
            :key="feedOnestopId"
            type="is-success"
            has-icon
            icon="information"
            :closable="false"
          >
            <div class="columns">
              <div class="column is-8">
                <p>
                  This operator includes data from the <strong>{{ feedSpec.replace('_', '-') }}</strong> feed record with Onestop ID of
                  <code>{{ feedOnestopId }}</code> See the feed record for Transitland's archive of fetched versions, as well as URLs for accessing the feed. <!-- TODO: show different text depending upon feed.spec = GTFS, GTFS-RT, or GBFS -->
                </p>
              </div>
              <div class="column is-4 has-text-right">
                <nuxt-link class="button is-primary" :to="{name:'feeds-feed', params:{feed:feedOnestopId}}">
                  View Feed Record
                </nuxt-link>
              </div>
            </div>
          </b-message>
        </b-tab-item>

        <b-tab-item label="Source Feeds (Advanced View)">
          <b-message type="is-light" has-icon icon="information" :closable="false">
            This operator includes data from the references listed below. These references are defined in the operator's Atlas record, and describe the GTFS agencies that provide the routes, stops, schedules, and other information for this operator. If a reference to an agency cannot be resolved, this will be noted. Please see the <nuxt-link :to="{name:'documentation'}">
              Operator documentation
            </nuxt-link> for more information on this process.
          </b-message>
          <div class="content">
            <b-table
              :data="sources"
              :striped="true"
              sort-icon="menu-up"
            >
              <b-table-column v-slot="props" label="Association type">
                {{ props.row.target_type }}
              </b-table-column>
              <b-table-column v-slot="props" label="Source Feed Onestop ID">
                <nuxt-link :to="{name:'feeds-feed', params:{feed:props.row.target_feed}}">
                  {{ props.row.target_feed }}
                </nuxt-link>
              </b-table-column>
              <b-table-column v-slot="props" label="Feed Spec">
                {{ props.row.target_feed_spec }}
              </b-table-column>
              <b-table-column v-slot="props" field="agency" label="Matched GTFS Agency">
                <template v-if="props.row.target_match">
                  <b-icon icon="check" />
                  {{ props.row.target_match.agency_name }}
                </template>
                <template v-else-if="props.row.feed_spec == 'GTFS'">
                  <b-tooltip dashed label="The active version of this source feed does not contain a matching agency">
                    <b-icon icon="alert" />
                  </b-tooltip>
                </template>
              </b-table-column>
            </b-table>
          </div>
        </b-tab-item>
      </b-tabs>

      <hr>

      <!-- anchors for when users click between tabs -->
      <div v-if="agencyIds.length > 0">
        <a v-for="[, value] of Object.entries(tabIndex)" :key="value" :name="value" />
        <h4 class="title is-4">
          Operator Service
        </h4>
        <b-tabs v-model="activeTab" type="is-boxed" :animated="false" @input="setTab">
          <b-tab-item label="Map">
            <client-only placeholder="Map">
              <tl-feed-version-map-viewer v-if="activeTab === 0" :agency-ids="agencyIds" :overlay="true" :link-version="linkVersion" />
            </client-only>
          </b-tab-item>

          <b-tab-item label="Routes">
            <tl-route-viewer v-if="activeTab === 1" :agency-ids="agencyIds" :show-agency="true" />
          </b-tab-item>

          <b-tab-item label="Stops">
            <tl-stop-viewer v-if="activeTab === 2" :agency-ids="agencyIds" />
          </b-tab-item>

          <b-tab-item v-if="advancedMode" label="Export">
            <template v-if="activeTab === 3 && agencyIds.length === 1">
              <client-only>
                <agency-export :agency-ids="agencyIds" />
              </client-only>
            </template>
            <template v-else>
              Currently this feature is only available when a single agency is returned for this query.
            </template>
          </b-tab-item>
        </b-tabs>
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'
import EntityPageMixin from './entity-page-mixin'

const q = gql`
query ($onestop_id: String, $feed_onestop_id: String) {
  entities: operators(limit: 10, where: {feed_onestop_id: $feed_onestop_id, onestop_id: $onestop_id, merged: true}) {
    id
    onestop_id
    generated
    file
    name
    short_name
    tags
    feeds {
      onestop_id
      spec
    }
    agencies {
      id
      feed_version_sha1
      feed_onestop_id
      agency_id
      agency_name
      agency_url
      agency_phone
      feed_version {
        id
        fetched_at
        feed {
          id
          onestop_id
          spec
        }
      }
      places(where: {min_rank: 0.2}) {
        city_name
        adm0_name
        adm1_name
      }
    }
  }
}
`
export default {
  mixins: [EntityPageMixin],
  data () {
    return {
      features: [],
      tabIndex: {
        0: 'map',
        1: 'routes',
        2: 'stops',
        3: 'export'
      }
    }
  },
  apollo: {
    entities: {
      client: 'transitland',
      query: q,
      // skip () { return this.checkSearchSkip(this.$route.query.agency_id) }, // skip if search and no agency_id
      variables () {
        return this.searchKey
      }
    }
  },
  head () {
    return {
      title: this.staticTitle,
      meta: [
        { hid: 'description', name: 'description', content: this.staticDescription },
        { hid: 'twitter:card', name: 'twitter:card', content: 'summary' },
        { hid: 'twitter:site', name: 'twitter:site', content: '@transitland' },
        { hid: 'twitter:title', name: 'twitter:title', content: this.staticTitle },
        { hid: 'twitter:image', name: 'twitter:image', content: 'https://www.transit.land/images/transitland-logo-square-with-whitebackground-smaller.png' },
        { hid: 'twitter:image:alt', name: 'twitter:image:alt', content: 'Transitland' },
        { hid: 'twitter:description', name: 'twitter:description', content: this.staticDescription },
        { hid: 'og:title', property: 'og:title', content: this.staticTitle },
        { hid: 'og:description', property: 'og:description', content: this.staticDescription }
      ]
    }
  },
  computed: {
    dataFreshness () {
      // The fetched_at is on agencies, not the top level entities
      const daysAgo = []
      const n = new Date()
      try {
        for (const ent of this.agencies) {
          const n2 = Date.parse(ent.feed_version.fetched_at)
          daysAgo.push(Math.floor((n2 - n) / (1000 * 3600 * 24 * -1)))
        }
      } catch {
      }
      return Math.max(...daysAgo)
    },
    locations () {
      const ret = new Map()
      for (const ent of this.agencies) {
        if (!ent) { continue }
        for (const p of ent.places || []) {
          const key = `${p.adm0_name} / ${p.adm1_name} - ${p.city_name}`
          ret.set(key, p)
        }
      }
      return Array.from(ret.values()).sort(function (a, b) { return a.rank - b.rank })
    },
    agencies () {
      return (this.entity || {}).agencies || []
    },
    agencyIds () {
      return this.agencies.map((s) => { return s.id }).filter((s) => { return s })
    },
    agencyNames () {
      return [...new Set(this.agencies.map((s) => { return s.agency_name }))]
    },
    agencyURLs () {
      return [...new Set(this.agencies.map((s) => { return s.agency_url }))]
    },
    generatedOperator () {
      return this.entity && this.entity.generated
    },
    operatorName () {
      if (this.entity && this.entity.name && this.entity.short_name) {
        return `${this.entity.name} (${this.entity.short_name})`
      } else if (this.entity && (this.entity.name || this.entity.short_name)) {
        return this.entity.name || this.entity.short_name
      } else if (this.agencies && this.agencies.length > 0) {
        return this.agencies[0].agency_name
      } else {
        return ''
      }
    },
    sources () {
      const ret = []
      const matchedFeeds = {}
      if (!this.entity) {
        return ret
      }
      for (const agency of this.agencies) {
        ret.push({
          target_type: 'Associated Feed',
          target_feed: agency.feed_version.feed.onestop_id,
          target_feed_spec: agency.feed_version.feed.spec.toUpperCase(),
          target_match: agency
        })
        matchedFeeds[agency.feed_version.feed.onestop_id] = true
      }
      if (this.entity) {
        for (const oif of this.entity.feeds || []) {
          const fid = oif.onestop_id
          if (!matchedFeeds[fid]) {
            ret.push({
              target_type: 'Associated Feed',
              target_feed: fid,
              target_feed_spec: oif.spec.toUpperCase()
            })
          }
        }
      }
      if (ret.length === 0) {
        ret.push({
          target_type: 'No Associations'
        })
      }
      return ret
    },
    uniqueFeedSourcesOnestopIds () {
      const onestopIdsToSpec = {}
      this.sources.forEach((s) => {
        onestopIdsToSpec[s.target_feed] = s.target_feed_spec.toUpperCase()
      })
      return onestopIdsToSpec
    },
    feedCounts () {
      return Object.keys(this.uniqueFeedSourcesOnestopIds).length
    },
    staticTitle () {
      return `${this.operatorName} • Operator details`
    },
    staticDescription () {
      const gtfsCount = this.sources.filter(s => s.target_feed_spec === 'GTFS').length
      const gtfsRtCount = this.sources.filter(s => s.target_feed_spec === 'GTFS_RT').length
      let feedCounts = `${gtfsCount} GTFS feed${gtfsCount > 1 ? 's' : ''}`
      if (gtfsRtCount > 0) {
        feedCounts += ` and ${gtfsRtCount} GTFS Realtime feed${gtfsRtCount > 1 ? 's' : ''}`
      }
      const locations = this.locations
        .map(l => [l.adm0_name, l.adm1_name, l.city_name].filter(Boolean).join(', '))
        .join('; ')
      return `${this.operatorName} is an operator listed on the Transitland open data platform. Transitland sources data for this operator from ${feedCounts}. ${this.operatorName} provides transit services in the following locations: ${locations}.`
    }
  }
}
</script>
