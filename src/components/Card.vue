<template>
  <v-card class="mx-auto" style="cursor: pointer" max-height="250" max-width="200">
    <v-img class="white--text align-end" height="100px" contain :src="thumbnailUrl"> </v-img>
    <v-card-title id="title" class="text-truncate">{{ pkg.name }}</v-card-title>
    <v-card-subtitle class="pb-0">{{
      $store.getters["authors/getAuthorNameById"](pkg.authorId)
    }}</v-card-subtitle>
    <v-card-actions>
      <v-btn
        block
        :loading="isLoading"
        :color="buttonConfig.color"
        dark
        @click="e => installActionHandlerWrapper(e, pkg, buttonConfig.handler)"
      >
        <template v-slot:loader>
          <v-progress-linear
            v-model="progressValue"
            color="accent"
            absolute
            bottom
            rounded
            height="100%"
            :indeterminate="isIndeterminate"
          >
          </v-progress-linear>
        </template>
        {{ buttonConfig.text }}
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
import { ipcRenderer } from "electron";
import { Vue, Component, Prop } from "vue-property-decorator";
import { getButtonConfig } from "../utils/install";
import { Package } from "types/package";

@Component
export default class Card extends Vue {
  @Prop() readonly pkg!: Package;

  // DATA PROPERTIES
  isLoading = false;
  isIndeterminate = true;
  progressValue = 0;

  // COMPUTED PROPERTIES
  get buttonConfig() {
    return getButtonConfig(this.pkg);
  }

  get thumbnailUrl() {
    if (this.pkg.images.length) {
      return this.pkg.images[0];
    } else
      return "https://avatars0.githubusercontent.com/in/88051?s=120&u=447b1928428587566a78aa1aadba9283685b23e4&v=4";
  }

  // METHODS
  async installActionHandlerWrapper(event: Event, pkg: Package, handler: Function) {
    this.isLoading = true;
    this.isIndeterminate = true;

    ipcRenderer.on("download-total", (_event, dlTotalBytes) => {
      if (dlTotalBytes > 2000000) this.isIndeterminate = false;
    });
    ipcRenderer.on("download-progress", (_event, dlPercent) => {
      this.progressValue = dlPercent * 100;
    });

    await handler(event, pkg);
    this.isIndeterminate = true;
    this.isLoading = false;
  }
}
</script>
<style lang="scss" scoped>
#title {
  display: block;
}
</style>
