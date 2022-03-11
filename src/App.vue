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
        <li v-for="(item, index) in navigationHistoryStack" :key="index">
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
        <li v-for="(item, index) in navigationStack" :key="item.id">
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

const FileList: FileDto[] = [
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

export class FileDto {
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
      listMessage: new Array<FileDto>(),
      checkMessage: {},
      navigationStack: new Array<FileDto>(),
      navigationHistoryStack: new Array<FileDto>(),
    };
  },

  created() {
    (this.checkMessage as any).parentId = null;

    this.gotoList();

    var rootFolder = new FileDto();
    rootFolder.fileName = "我的网盘";
    this.navigationStack.push(rootFolder);
    this.pushNavigationHistoryStack(rootFolder);
  },

  methods: {
    open(item: FileDto) {
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

    toFolder(folder: FileDto) {
      if ((this.checkMessage as any).parentId == folder.id) {
        return false;
      }

      (this.checkMessage as any).parentId = folder.id;

      this.gotoList();
      return true;
    },

    navigationBack() {
      if (this.navigationStack.length == 1) {
        return;
      }
      this.navigationStack.pop();
      var lastItem = Enumerable.from(this.navigationStack).lastOrDefault();
      if (lastItem == null) {
        return;
      }
      if (this.toFolder(lastItem)) {
        this.navigationHistoryStack.forEach((element) => {
          element["isCurrent"] = false;
        });
        lastItem["isCurrent"] = true;

        this.pushNavigationHistoryStack(lastItem);
      }
    },
    pushNavigationHistoryStack(item) {
      var newItem = Object.assign({}, item);

      if (this.navigationHistoryStack.length > 10) {
        this.navigationHistoryStack.pop();
      }
      this.navigationHistoryStack.unshift(newItem);
    },

    dealWithNavigationStack(folder) {
      var toIndex = Enumerable.from(this.navigationStack).indexOf(
        (c) => c.id == folder.id
      );
      if (toIndex >= 0) {
        this.navigationStack.splice(
          toIndex + 1,
          this.navigationStack.length - toIndex - 1
        );
      } else {
        this.navigationStack.push(folder);
      }
    },

    navigationTo(folder) {
      this.dealWithNavigationStack(folder);
      if (this.toFolder(folder)) {
        this.navigationHistoryStack.forEach((element) => {
          element["isCurrent"] = false;
        });
        folder["isCurrent"] = true;
        this.pushNavigationHistoryStack(folder);
      }
    },

    navigationHistoryBack() {
      var currentIndex = Enumerable.from(this.navigationHistoryStack).indexOf(
        (c) => c["isCurrent"]
      );
      if (currentIndex < this.navigationHistoryStack.length - 1) {
        var forwardIndex = currentIndex + 1;

        var folder = this.navigationHistoryStack[forwardIndex];
        this.dealWithNavigationStack(folder);

        if (this.toFolder(folder)) {
          this.navigationHistoryStack.forEach((element) => {
            element["isCurrent"] = false;
          });
          this.navigationHistoryStack[forwardIndex]["isCurrent"] = true;
        }
      }
    },

    navigationHistoryForward() {
      var currentIndex = Enumerable.from(this.navigationHistoryStack).indexOf(
        (c) => c["isCurrent"]
      );
      if (currentIndex > 0) {
        var forwardIndex = currentIndex - 1;

        var folder = this.navigationHistoryStack[forwardIndex];
        this.dealWithNavigationStack(folder);

        if (this.toFolder(folder)) {
          this.navigationHistoryStack.forEach((element) => {
            element["isCurrent"] = false;
          });
          this.navigationHistoryStack[forwardIndex]["isCurrent"] = true;
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
