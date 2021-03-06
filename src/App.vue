<template>
  <v-app>
    <v-app-bar app color="grey darken-4" dark class="header">
      <!-- <v-app-bar-nav-icon @click="isOpen = !isOpen" /> -->

      <div
        class="d-flex align-center headline ml-5 font-weight-light"
        style="letter-spacing:0.2em !important"
      >
        気象庁防災情報 XML Viewer
      </div>

      <v-spacer />

      <v-btn color="teal" outlined @click="Open">
        <v-icon>mdi-open-in-new</v-icon>
        <span class="ml-2">XML を開く</span>
      </v-btn>

      <v-btn class="ml-5" color="warning" :loading="isRefresh" outlined @click="Refresh">
        <v-icon>mdi-refresh</v-icon>
        <span class="ml-2">データ更新</span>
      </v-btn>

      <v-btn class="ml-5" color="primary" @click="ShowTitleList">
        <v-icon>mdi-format-list-bulleted-square</v-icon>
        <span class="ml-2">電文種別: {{ title }}</span>
      </v-btn>
    </v-app-bar>

    <v-content :class="listView ? 'v-content-darken' : ''">
      <Main ref="main" @UpdateRefreshState="UpdateRefreshState" @url="xmlUrl = $event" />
    </v-content>

    <div class="titleList" :class="listView ? 'titleList-active' : ''">
      <div class="titleList-name">電文種別選択</div>
      <div class="titleList-box">
        <div v-for="(v, k) in titles" :key="k" class="titleList-item" @click="ChangeTitle(k)">
          {{ v }}
        </div>
      </div>
    </div>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue"
import axios from "axios"
import Main from "@/components/Main.vue"

export default Vue.extend({
  name: "App",
  components: {
    Main
  },
  data: () => ({
    title: "すべて",
    titles: [""],
    listView: false,
    isRefresh: false,
    xmlUrl: ""
  }),
  async mounted() {
    this.titles = await (async () =>
      (await axios.get<string[]>("https://api.jmx.app/titles.json")).data)()
  },
  methods: {
    ShowTitleList() {
      this.listView = !this.listView
    },
    ChangeTitle(k: number) {
      this.listView = !this.listView
      this.title = this.titles[k]
      // eslint-disable-next-line no-extra-parens
      ;(this.$refs.main as InstanceType<typeof Main>).LoadList(this.title)
    },
    Open() {
      window.open(this.xmlUrl)
    },
    Refresh() {
      this.isRefresh = true
      // eslint-disable-next-line no-extra-parens
      ;(this.$refs.main as InstanceType<typeof Main>).LoadList(this.title)
    },
    UpdateRefreshState(x: boolean) {
      this.isRefresh = x
    }
  }
})
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Noto+Sans+JP&display=swap");

html {
  overflow: hidden !important;
}

body {
  position: relative;
  overflow: hidden !important;
}

* {
  font-family: "Noto Sans JP", sans-serif;
}

.v-toolbar__content {
  height: 64px !important;
  z-index: 9999;
}

.v-content {
  position: absolute;
  padding-top: 64px !important;
  top: 0;
  left: 0;
  z-index: 1;

  &::after {
    content: "";
    position: absolute;
    width: 100%;
    height: calc(100vh - 64px);
    top: 64px;
    left: 0;
    opacity: 0;
    background-color: #000000;
    transition: 0.3s all;
    z-index: -1;
  }

  &-darken {
    &::after {
      opacity: 0.75;
      z-index: 9998;
    }
  }
}
</style>

<style lang="scss" scoped>
.header {
  box-sizing: border-box;
  height: 64px !important;
  z-index: 9999;
  cursor: default;
}

.titleList {
  box-sizing: border-box;
  padding: 30px;
  position: absolute;
  width: calc(100vw - 40px);
  height: calc(100vh - 104px);
  top: 164px;
  left: 20px;
  background-color: #fff;
  opacity: 0;
  border-radius: 20px;
  will-change: top;
  transition: 0.3s all;
  cursor: default;
  z-index: -1;

  &-active {
    top: 84px;
    z-index: 999;
    opacity: 1;
  }

  &-name {
    padding: 10px 0;
    font-size: 30px;
    font-weight: bold;
    text-align: center;
    letter-spacing: 0.2em;
  }

  &-box {
    display: flex;
    padding-bottom: 15px;
    height: calc(100vh - 229px);
    justify-content: center;
    flex-wrap: wrap;
    overflow: auto;
  }

  &-item {
    display: flex;
    margin: 15px 7.5px 0;
    padding: 8px 15px;
    width: 360px;
    outline: 1px solid #cdcdcd;
    text-align: center;
    transition: 0.3s all;
    background-color: #ffffff;
    cursor: pointer;
    justify-content: center;
    align-items: center;

    &:hover {
      background-color: #eeeeee;
    }
  }
}
</style>
