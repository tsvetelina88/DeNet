<template>
  <div>
    <q-layout view="lHh Lpr lff">
      <!-- Drawer for the dock panel -->
      <q-drawer show-if-above v-model:drawer="drawer" side="left" bordered>
        <q-list>
          <q-item
            v-for="(site, index) in sites"
            :key="index"
            @click="openSite(site.url)"
          >
            <q-item-section>
              {{ site.name }}
            </q-item-section>
          </q-item>
        </q-list>
      </q-drawer>

      <!-- Main content area -->
      <q-page-container>
        <q-btn @click="drawer = !drawer" color="primary" class="q-mb-md">
          Toggle Dock Panel
        </q-btn>
        <q-page class="q-pa-md">
          <iframe
            :src="currentSite"
            width="100%"
            height="600"
            frameborder="0"
          ></iframe>
        </q-page>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

// Define sites for the dock panel
const sites = ref([
  { name: "Uniswap", url: "https://app.uniswap.org/" },
  { name: "1inch", url: "https://app.1inch.io/" },
  { name: "Bugs Denet", url: "https://bugs.denet.pro" },
  { name: "Revert Finance", url: "https://revert.finance" },
  { name: "Google", url: "https://www.google.com" }, // Added extra site for demonstration
]);

// Reactive variable to control the drawer
const drawer = ref(false);

// Reactive variable to hold the URL of the currently selected site
const currentSite = ref("");

// Function to set the URL of the iframe based on the selected site
const openSite = (url: string) => {
  currentSite.value = url;
};
</script>

<style scoped>
/* Optional: Add some styles if needed */
iframe {
  border: none;
}
</style>
