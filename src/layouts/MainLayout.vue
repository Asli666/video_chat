<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Video Chat Demo
        </q-toolbar-title>

        <div>Author: 李艾珊，赵冬阳，赵通</div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>


export default {
  name: 'MainLayout',
  // components: { EssentialLink },
  data () {
    return {
      leftDrawerOpen: false,
      // essentialLinks: linksData
    }
  },

  mounted: function () {
  this.$nextTick(function () {
  const s = document.createElement('script');
  s.type = 'text/javascript';
  s.src = 'https://cdn.scaledrone.com/scaledrone.min.js';
  document.body.appendChild(s);
  })
  }


}
</script>
