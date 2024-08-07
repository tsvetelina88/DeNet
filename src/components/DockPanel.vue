<script setup lang="ts">
import { ref, onMounted, defineProps } from "vue";
import {
  QBtn,
  QLayout,
  QPage,
  QPageContainer,
  QList,
  QDrawer,
  QItem,
  QItemSection,
} from "Quasar";

const props = defineProps<{
  jsonFile?: string;
  appList?: { name: string; url: string }[];
}>();

const drawer = ref(false);
const currentSite = ref("");
const sites = ref<{ name: string; url: string }[]>([]);

const openSite = (url: string) => {
  currentSite.value = url;
};

const toggleDrawer = () => {
  drawer.value = !drawer.value;
};

onMounted(async () => {
  if (props.appList) {
    sites.value = props.appList;
  } else if (props.jsonFile) {
    try {
      const response = await fetch(props.jsonFile);
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      sites.value = await response.json();
    } catch (error) {
      console.error("Failed to load applications:", error);
    }
  } else {
    console.warn("No application list provided.");
  }

  if (sites.value.length > 0) {
    currentSite.value = sites.value[0].url;
  }
});
</script>
<template>
  <q-layout view="lHh Lpr lff">
    <q-drawer v-model="drawer" side="left" bordered>
      <q-list>
        <q-item v-for="(site, index) in sites" :key="index" class="q-mr-md">
          <q-item-section
            style="color: red"
            clickable
            @click="openSite(site.url)"
          >
            {{ site.name }}
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <q-btn @click="toggleDrawer" color="primary" class="q-mb-md">
        Show Menu
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
</template>

<style scoped>
iframe {
  border: 1px solid red;
}
</style>
