<template>
  <div class="preview">
    <topology preview="true" :data="data" />
    <div class="tools" v-if="showTools">
      <a-button type="primary" @click="onBack">
        <a-icon type="left" /> 返回
      </a-button>
      <div></div>
      <a-button type="primary" style="margin-right: 0" @click="onSizeWindow">
        <a-icon type="border" />
        适合窗口大小
      </a-button>
      <a-button type="primary" @click="onSizeOri">
        <a-icon type="fullscreen-exit" />
        实际大小
      </a-button>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import Topology from 'topology-vue';
Vue.use(Topology);
import axios from 'axios';
export default {
  name: 'Preview',
  data() {
    return {
      data: { data: {} },
      locked: 0,
      showTools: true,
    };
  },
  watch: {
    $route() {
      this.init();
    },
  },
  created() {
    this.init();
    const data = window.topologyData;
    if (data) {
      this.locked = data.locked;
      data.locked = 1;
      this.$set(this.data, 'data', Object.assign({}, data));
      setTimeout(() => {
        window.topologyData = null;
      }, 200);
    } else {
      // Do sth.
    }

    this.showTools = !!this.$route.query.r;
  },
  methods: {
    onSizeWindow() {
      window.topology.fitView(16);
    },

    onSizeOri() {
      window.topology.open(this.data.data);
    },

    onBack() {
      window.topology.data.locked = this.locked;
      window.topologyData = window.topology.data;
      this.$router.go(-1);
    },

    async init() {
      if (this.$route.query.id) {
        let ret = await axios.get('/api/topology/' + this.$route.query.id, {
          params: {
            version: this.$route.query.version,
            view: 1,
          },
        });
        if (ret.error) {
          return;
        }

        if (!ret.pens) {
          const data = ret.data;
          delete ret.data;
          ret = Object.assign(ret, data);
        }
        this.data = ret;
      } else {
        this.data = window.topologyData || {};
        setTimeout(() => {
          window.topologyData = null;
        }, 200);
      }

      this.showTools = !!this.$route.query.r;
    },
  },
};
</script>
<style lang="scss" scoped>
.preview {
  position: relative;
  height: 100vh;

  .tools {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    display: flex;
    & > div {
      flex-grow: 1;
    }

    button {
      margin: 12px 16px;
    }
  }
}
</style>
