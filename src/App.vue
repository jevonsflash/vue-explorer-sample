<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" /> -->

    <div class="buttons">
      <div>
        <button @click="navigationHistoryBack">
          <img
            style="transform: rotate(180deg)"
            :src="require('@/assets/arr.png')"
          />
        </button>
      </div>
      <div>
        <button @click="navigationHistoryForward">
          <img :src="require('@/assets/arr.png')" />
        </button>
      </div>
      <div>
        <a @click="show">
          <img
            :src="require('@/assets/arr2.png')"
            :style="{
              transform: showHistory ? 'rotate(0deg)' : 'rotate(-180deg)',
            }"
          />
        </a>
      </div>
      <ul class="history" v-show="showHistory">
        <li v-for="(item, index) in NavigationHistoryStack" :key="index">
          <span>{{ item.fileName }}</span>

          <span v-if="getIsCurrentHistoryNavigationItem(item)"> (当前)</span>
        </li>
      </ul>

      <div>
        <button @click="navigationBack">
          <img
            style="transform: rotate(-90deg)"
            :src="require('@/assets/arr.png')"
          />
        </button>
      </div>
    </div>

    <div class="crumbs">
      <ul>
        <li v-for="(item, index) in NavigationStack" :key="item.id">
          {{ index > 0 ? " /" : "" }}
          <a href="javascript:void(0)" @click="navigationTo(item)">{{
            item.fileName
          }}</a>
        </li>
      </ul>
    </div>
    <div class="file-content">
      <table border="1">
        <tr>
          <th>id</th>
          <th>文件名</th>
          <th>类型</th>
          <th>大小</th>
        </tr>
        <tr v-for="item in listMessage" :key="item.id">
          <td>{{ item.id }}</td>
          <td>
            <a href="javascript:void(0)" @click="open(item)">{{
              item.fileName
            }}</a>
          </td>
          <td>{{ item.fileType == 1 ? "目录" : "文件" }}</td>
          <td>{{ item.fileType == 1 ? "/" : `${item.byteSize}M` }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script lang='ts'>
import Enumerable from "linq";
import HelloWorld from './components/HelloWorld.vue';

const FileList: FileBriefWithThumbnailDto[] = [
  {
    id: 1,
    parentId: null,
    fileName: "文件1",
    fileType: 2,
    byteSize: 1024,
  },
  {
    id: 2,
    parentId: null,
    fileName: "文件2",
    fileType: 2,
    byteSize: 1024,
  },

  {
    id: 3,
    parentId: null,
    fileName: "文件夹A",
    fileType: 1,
    byteSize: null,
  },
  {
    id: 4,
    parentId: 3,
    fileName: "文件夹B",
    fileType: 1,
    byteSize: null,
  },
  {
    id: 5,
    parentId: 3,
    fileName: "文件夹C",
    fileType: 1,
    byteSize: null,
  },
  {
    id: 6,
    parentId: 3,
    fileName: "文件3",
    fileType: 2,
    byteSize: 1024,
  },
  {
    id: 7,
    parentId: 5,
    fileName: "文件夹D",
    fileType: 1,
    byteSize: null,
  },
];

export class FileBriefWithThumbnailDto {
  id: number;
  parentId: number;
  fileName: string;
  fileType: number;
  byteSize: number;
}

export default {
  name: "App",
  // components: {HelloWorld},
  data: () => {
    return {
      showHistory: false,
      listMessage: new Array<FileBriefWithThumbnailDto>(),
      checkMessage: {},
      NavigationStack: new Array<FileBriefWithThumbnailDto>(),
      NavigationHistoryStack: new Array<FileBriefWithThumbnailDto>(),
    };
  },

  created() {
    (this.checkMessage as any).parentId = null;

    this.gotoList();

    var rootFolder = new FileBriefWithThumbnailDto();
    rootFolder.fileName = "我的网盘";
    this.NavigationStack.push(rootFolder);
    this.pushNavigationHistoryStack(rootFolder);
  },

  methods: {
    open(item: FileBriefWithThumbnailDto) {
      if (item.fileType == 1) {
        this.navigationTo(item);
      } else {
        window.alert("打开了文件" + item.fileName);
      }
    },

    gotoList() {
      this.listMessage = Enumerable.from(FileList)
        .where((c) => c.parentId == (this.checkMessage as any).parentId)
        .toArray();
    },
    show() {
      this.showHistory = !this.showHistory;
    },

    toFolder(folder: FileBriefWithThumbnailDto) {
      if ((this.checkMessage as any).parentId == folder.id) {
        return false;
      }

      (this.checkMessage as any).parentId = folder.id;

      this.gotoList();
      return true;
    },

    navigationBack() {
      if (this.NavigationStack.length == 1) {
        return;
      }
      this.NavigationStack.pop();
      var lastItem = Enumerable.from(this.NavigationStack).lastOrDefault();
      if (lastItem == null) {
        return;
      }
      if (this.toFolder(lastItem)) {
        this.NavigationHistoryStack.forEach((element) => {
          element["isCurrent"] = false;
        });
        lastItem["isCurrent"] = true;

        this.pushNavigationHistoryStack(lastItem);
      }
    },
    pushNavigationHistoryStack(item) {
      var newItem = Object.assign({}, item);

      if (this.NavigationHistoryStack.length > 10) {
        this.NavigationHistoryStack.pop();
      }
      this.NavigationHistoryStack.unshift(newItem);
    },

    dealWithNavigationStack(folder) {
      var toIndex = Enumerable.from(this.NavigationStack).indexOf(
        (c) => c.id == folder.id
      );
      if (toIndex >= 0) {
        this.NavigationStack.splice(
          toIndex + 1,
          this.NavigationStack.length - toIndex - 1
        );
      } else {
        this.NavigationStack.push(folder);
      }
    },

    navigationTo(folder) {
      this.dealWithNavigationStack(folder);
      if (this.toFolder(folder)) {
        this.NavigationHistoryStack.forEach((element) => {
          element["isCurrent"] = false;
        });
        folder["isCurrent"] = true;
        this.pushNavigationHistoryStack(folder);
      }
    },

    navigationHistoryBack() {
      var currentIndex = Enumerable.from(this.NavigationHistoryStack).indexOf(
        (c) => c["isCurrent"]
      );
      if (currentIndex < this.NavigationHistoryStack.length - 1) {
        var forwardIndex = currentIndex + 1;

        var folder = this.NavigationHistoryStack[forwardIndex];
        this.dealWithNavigationStack(folder);

        if (this.toFolder(folder)) {
          this.NavigationHistoryStack.forEach((element) => {
            element["isCurrent"] = false;
          });
          this.NavigationHistoryStack[forwardIndex]["isCurrent"] = true;
        }
      }
    },

    navigationHistoryForward() {
      var currentIndex = Enumerable.from(this.NavigationHistoryStack).indexOf(
        (c) => c["isCurrent"]
      );
      if (currentIndex > 0) {
        var forwardIndex = currentIndex - 1;

        var folder = this.NavigationHistoryStack[forwardIndex];
        this.dealWithNavigationStack(folder);

        if (this.toFolder(folder)) {
          this.NavigationHistoryStack.forEach((element) => {
            element["isCurrent"] = false;
          });
          this.NavigationHistoryStack[forwardIndex]["isCurrent"] = true;
        }
      }
    },

    getIsCurrentHistoryNavigationItem(item) {
      var result = item["isCurrent"];
      return result;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
}
.buttons {
  display: inline-block;
}

.buttons img {
  width: 10px;
}

.buttons div {
  display: inline-block;
}

.history {
  position: absolute;
  display: block;
  background: #ffffffd4;
  box-shadow: 0px 8px 16px 0px rgb(0 0 0 / 20%);
  min-height: 120px;
  padding-right: 20px;
}

.crumbs {
  display: inline-block;
}

.crumbs ul li {
  display: inline-block;
}

.file-content table {
  width: 100%;
}
</style>
