<script lang="ts">
import { defineComponent } from "vue";
import axios from "axios";
import type { GHContents } from "@/types";

export default defineComponent({
  data() {
    return {
      contents: [] as GHContents[],
      basePath: "",
    };
  },
  methods: {
    async fetch() {
      const res = await axios({
        method: "get",
        url: `https://api.github.com/repos/loicbacciga/study-notes/contents/${this.basePath}`,
        headers: {
          Accept: "application/vnd.github+json",
          Authorization: `Bearer ${import.meta.env.VITE_GH_TOKEN}`,
        },
      });

      console.log(res.data);
      this.contents = res.data as GHContents[];
    },

    goTo(index: number) {
      this.basePath = this.basePath
        .split("/")
        .slice(0, index + 1)
        .join("/");
    },
  },
  watch: {
    basePath() {
      this.fetch();
    },
  },

  created() {
    this.fetch();
  },
});
</script>

<template>
  <main>
    <p>Path: "{{ basePath }}"</p>

    <p>
      <a @click="basePath = ''">Home</a>
      <a
        v-show="basePath.length > 0"
        v-for="[i, path] in basePath.split('/').entries()"
        :key="path"
        @click="goTo(i)"
      >
        -
        {{ path }}
      </a>
    </p>
    <ul>
      <li v-for="content in contents" :key="content.name">
        <button @click="basePath = content.path">
          {{ content.name }} | {{ content.type }}
          <iframe
            v-if="content.type === 'file'"
            :src="`https://docs.google.com/gview?url=${content.download_url}f&embedded=true`"
            style="width: 600px; height: 500px"
            frameborder="0"
          ></iframe>
        </button>
      </li>
    </ul>
  </main>
</template>
