<template>
  <div class="home">
    <!-- 使用topology组件 -->
    <topology
      :configs="topologyConfigs"
      :materials="materials"
      :user="user"
      :data="data"
      @event="onEvent"
    />
  </div>
</template>

<script>
import Vue from 'vue';
// 导入topology-vue组件
import topology from 'topology-vue';
// 需要导入topology-vue.css
import 'topology-vue/topology-vue.css';
import axios from 'axios';
Vue.use(topology);
import {
  defalutMenus,
  defalutUserMenus,
  // defalutMaterials,
  // userMaterials,
  // images,
} from './data';

export default {
  name: 'Home',
  data: function () {
    return {
      topologyConfigs: {
        license: {
          key: '企业版授权码',
          value: '国产原创开源发展做出我们的贡献',
        },
        logo: {
          img: 'http://topology.le5le.com/assets/img/favicon.ico',
          url: 'http://topology.le5le.com',
          target: '_blank',
        },
        menus: defalutMenus,
        loginUrl: 'https://account.le5le.com',
        signupUrl: 'https://account.le5le.com',
        userMenus: defalutUserMenus,
        dataOptionsFn: (pen, key, value) => {
          // pen - 当前画笔对象
          // key - 表单输入左侧的属性名
          // value - 仅下拉搜索时有效，当前输入文本
          console.log('dataOptionsFn', pen, key, value);
          // ************
          // 根据实际业务场景 + 参数返回数组对象。
          // Do sth.
          // ************
          return [
            {
              label: '选项1',
              value: 1,
            },
            {
              label: '选项2',
              value: 2,
            },
            {
              label: '选项3',
              value: 3,
            },
          ];
        },
      },
      user: {
        username: 'le5le',
      },
      materials: {
        topology: [],
        system: [],
        user: [],
        uploadUrl: '/api/image',
        uploadHeaders: {
          Authorization: 'your token',
        },
        uploadParams: {
          mydata: 1,
        },
      },
      data: {},
    };
  },
  created: function () {
    const data = window.topologyData;
    // 存在缓存数据，预览页返回
    if (data) {
      this.data = { data: Object.assign({}, data) };
      setTimeout(() => {
        window.topologyData = null;
      }, 200);
    } else {
      // Do sth.
    }
    this.getMaterials();

    this.init();
  },
  watch: {
    $route() {
      this.init();
    },
  },
  methods: {
    async getMaterials() {
      let folder = await axios.get('/api/user/folder');
      if (folder.data.error) {
        folder = {};
      } else {
        folder = folder.data;
      }

      if (folder.topology) {
        this.materials.topology = [];
        folder.topology.forEach((name) => {
          this.materials.topology.push({
            name,
            show: true,
            expand: true,
            list: [],
          });
        });
      }

      if (folder.user) {
        this.materials.user = [];
        folder.user.forEach((name) => {
          this.materials.user.push({
            name,
            show: true,
            expand: true,
            list: [],
          });
        });
      }

      const tools = await axios.get('/api/tools');
      const group = {};
      tools.data.forEach((item) => {
        if (!group[item.subClassName]) {
          group[item.subClassName] = {
            name: item.subClassName,
            show: true,
            expand: true,
            list: [item],
          };
        } else {
          group[item.subClassName].list.push(item);
        }
      });

      this.materials.system = [];
      for (const key in group) {
        this.materials.system.push(group[key]);
      }

      const topologies = await axios.get('/api/topologies', {
        params: {
          pageIndex: 1,
          pageCount: 100,
          component: 'all',
        },
      });

      if (topologies.data.list) {
        this.materials.topology.push({
          name: '未分类',
          show: true,
          expand: true,
          list: [],
        });
        this.materials.user.push({
          name: '未分类',
          show: true,
          expand: true,
          list: [],
        });
        topologies.data.list.forEach((item) => {
          if (item.component) {
            this.materials.user.forEach((folder) => {
              if (item.folder === folder.name) {
                folder.list.push(item);
              }

              if (!item.folder && folder.name === '未分类') {
                folder.list.push(item);
              }
            });
          } else {
            this.materials.topology.forEach((folder) => {
              if (item.folder === folder.name) {
                folder.list.push(item);
              }

              if (!item.folder && folder.name === '未分类') {
                folder.list.push(item);
              }
            });
          }
        });
      }

      const componentImages = await axios.get('/api/user/component/images', {
        params: {
          pageIndex: 1,
          pageCount: 100,
        },
      });
      if (componentImages.data.list) {
        componentImages.data.list.forEach((item) => {
          this.materials.user.forEach((folder) => {
            if (item.folder === folder.name) {
              folder.list.push(item);
            }

            if (!item.folder && folder.name === '未分类') {
              folder.list.push(item);
            }
          });
        });
      }

      this.materials = Object.assign({}, this.materials);
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
        this.data = window.topologyData || {
          component: !!this.$route.query.component,
          folder: this.$route.query.folder,
        };

        setTimeout(() => {
          window.topologyData = null;
        }, 200);
      }
    },
    onEvent(e) {
      switch (e.name) {
        case 'add-topology-folder':
          this.addTopologyFolder(e.params);
          break;
        case 'rename-topology-folder':
          this.renameTopologyFolder(e.params);
          break;
        case 'remove-topology-folder':
          this.removeTopologyFolder(e.params);
          break;

        case 'add-user-folder':
          this.addUserFolder(e.params);
          break;
        case 'rename-user-folder':
          this.renameUserFolder(e.params);
          break;
        case 'remove-user-folder':
          this.removeUserFolder(e.params);
          break;
        case 'addImageUrl':
          this.addComponentImage(e.params);
          break;
        case 'deleteImage':
          this.deleteComponentImage(e.params);
          break;
        case 'logout':
          // 退出登录
          this.user = null;
          // Do sth.
          break;

        case 'openMaterialGroup':
          // 展开/折叠图标工具栏分组
          console.log('openMaterialGroup', e.params);
          // Do sth.
          break;

        case 't-topology-folder-open': // 打开文件夹
          this.openFolder(e.params);
          break;

        case 'addMaterial':
          // 添加“我的组件”
          // Do sth. For example:
          this.$router.push({
            path: '/',
            query: { folder: e.params, component: '1', r: Date.now() + '' },
          });
          break;

        case 'editMaterial':
          // 编辑“我的组件”
          // Do sth. For example:
          this.$router.push({
            path: '/',
            query: {
              id: e.params.id,
              folder: e.params,
              component: '1',
              r: Date.now() + '',
            },
          });
          break;

        case 'newFile':
          this.$router.push({
            path: '/',
            query: { folder: e.params, r: Date.now() + '' },
          });
          break;
        case 'save':
          this.save();
          break;

        case 'view':
          this.$router.push({
            path: '/',
            query: { id: e.params.id, r: Date.now() + '' },
          });
          break;
        case 'preview':
          // 点击工具栏“预览”
          // 保存当前编辑数据，方便预览时直接打开
          window.topologyData = window.topology.data;
          this.$router.push({
            path: '/preview',
            query: { id: this.$route.query.id, r: Date.now() + '' },
          });
          break;

        // ...
        // ...
      }
    },
    async openFolder(params) {
      const rep = await axios.get('/api/user/topologies', {
        params: {
          folder: params.folder,
          component: params.component,
          pageIndex: 1,
          pageCount: 100,
        },
      });
      // 对数据进行调整
      if (params.component) {
        // 组件
        let userFolder = this.materials.user?.find((item) => {
          return item.name === params.folder;
        });
        if (userFolder) {
          // 存在该文件夹，且数据不为空
          userFolder.list = rep.list ? rep.list : [];
          userFolder.loading = false;
        }
      } else {
        // 图纸
        let topologyFolder = this.materials.topology?.find(
          (item) => item.name === params.folder
        );
        if (topologyFolder) {
          // 存在该文件夹，且数据不为空
          topologyFolder.list = rep.list ? rep.list : [];
          topologyFolder.loading = false;
        }
      }
    },
    async addTopologyFolder(name) {
      let folder = await axios.post('/api/user/folder', {
        type: 'topology',
        name,
      });
      if (folder.data.error) {
        alert('新增文件夹失败：' + folder.data.error);
      }
    },

    async renameTopologyFolder(params) {
      params.type = 'topology';
      let folder = await axios.post('/api/user/folder', params);
      if (folder.data.error) {
        alert('修改文件夹失败：' + folder.data.error);
      }
    },

    async removeTopologyFolder(params) {
      params.type = 'topology';
      let folder = await axios.post('/api/user/folder/delete', params);
      if (folder.data.error) {
        alert('删除文件夹失败：' + folder.data.error);
      }
    },

    async addUserFolder(name) {
      let folder = await axios.post('/api/user/folder', {
        type: 'user',
        name,
      });
      if (folder.data.error) {
        alert('新增文件夹失败：' + folder.data.error);
      }
    },

    async renameUserFolder(params) {
      params.type = 'user';
      let folder = await axios.post('/api/user/folder', params);
      if (folder.data.error) {
        alert('修改文件夹失败：' + folder.data.error);
      }
    },

    async removeUserFolder(params) {
      params.type = 'user';
      let folder = await axios.post('/api/user/folder/delete', params);
      if (folder.data.error) {
        alert('删除文件夹失败：' + folder.data.error);
      }
    },

    save(tip = true) {
      window.topology.toImage(10, 'image/png', 1, async (blob) => {
        if (this.data.id) {
          if (!(await this.delImage(this.data.image))) {
            return;
          }
        }

        const file = await this.upload(blob, true);
        if (!file) {
          return;
        }

        this.data = Object.assign({}, window.topology.pureData());
        if (this.data.component) {
          this.data.componentData = window.topology.toComponent();
        }

        this.data.image = file.data.url;

        let ret;
        if (!this.data.name) {
          this.data.name = `topology.${new Date().toLocaleString()}`;
        }
        if (this.data.id) {
          ret = await axios.put('/api/user/topology', this.data);
        } else {
          ret = await axios.post('/api/user/topology', this.data);
        }

        if (ret.data.error) {
          return null;
        }

        if (!this.data.id || this.$route.query.version) {
          this.data.id = ret.data.id;
          this.$router.push({
            path: '/',
            query: { id: this.data.id },
          });
        }

        tip && alert('保存成功！');

        this.getMaterials();
      });
    },

    async upload(blob, shared = false, filename = '/topology/thumb.png') {
      const form = new FormData();
      form.append('path', filename);
      form.append('randomName', '1');
      form.append('public', shared + '');
      form.append('file', blob);
      const ret = await axios.post('/api/image', form);
      if (ret.data.error) {
        return null;
      }

      return ret;
    },

    async delImage(image) {
      const ret = await axios.delete('/api' + image);
      if (ret.data.error) {
        return false;
      }

      return true;
    },

    async addComponentImage(params) {
      const ret = await axios.post('/api/user/component/image', params);
      if (ret.error) {
        return false;
      }

      this.getMaterials();
    },

    async deleteComponentImage(params) {
      const ret = await axios.post('/api/user/component/image/delete', params);
      if (ret.error) {
        return false;
      }

      this.getMaterials();
    },
  },
};
</script>
<style lang="scss">
.home {
  height: 100vh;
}
</style>
