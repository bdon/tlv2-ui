<template>
  <b-navbar shadow>
    <template slot="brand">
      <nuxt-link class="navbar-item" :to="{name:'index'}">
        <div class="logo-image" />
      </nuxt-link>
    </template>
    <template slot="start">
      <client-only>
        <nuxt-link v-for="item in items" :key="item.title" :to="item.to" class="navbar-item" :class="($route && $route.name && $route.name.startsWith(item.to.name)) ? 'is-active is-tab': ''">
          {{ item.title }}
        </nuxt-link>
        <template slot="placeholder">
          <!-- when doing a static render, we cannot use nuxt-link or check $route, therefore we will temporarily render conventional links -->
          <a v-for="item in items" :key="item.title" class="navbar-item" :href="item.to.name === 'index' ? '/' : `/${item.to.name}`">{{ item.title }}</a>
        </template>
      </client-only>
    </template>
    <template slot="end">
      <b-navbar-item tag="div" :style="searchBarWidth">
        <tl-nav-search-bar
          style="width:100%"
          @focus="searchFocus = true"
          @blur="searchFocus = false"
        />
      </b-navbar-item>
    </template>
  </b-navbar>
</template>

<script>
export default {
  data () {
    return {
      searchFocus: false,
      apikey: null,
      items: [
        {
          title: 'Home',
          icon: 'home',
          to: { name: 'index' }
        },
        {
          title: 'Map',
          icon: 'lightbulb',
          to: { name: 'map' }
        },
        {
          title: 'Operators',
          icon: 'lightbulb',
          to: { name: 'operators' }
        },
        {
          title: 'Source Feeds',
          icon: 'lightbulb',
          to: { name: 'feeds' }
        },
        {
          title: 'Documentation',
          icon: 'lightbulb',
          to: { name: 'documentation' }
        }
      ]
    }
  },
  computed: {
    searchBarWidth () {
      return { 'flex-grow': this.searchFocus ? 1 : 0.25 }
    }
  }
}
</script>

<style scoped>
.navbar {
  background:#f9f9f9;
}
a {
  color: #000;
}
.logo-image {
    background: url('~assets/transitland_logo.svg');
    background-size: contain;
    background-repeat: no-repeat;
    margin-left:20px;
    margin-right:20px;
    width: 80px;
    height: 40px;
}
.notification {
  padding: 0.6em;
}
.notification p {
  display: block;
}
.client-only-placeholder {
  display: flex;
}
</style>
