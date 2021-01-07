<template>
  <div class="container">
    <webview
      id="printWebview"
      ref="printWebview"
      :src="fullPath"
      nodeintegration
    />
  </div>
</template>

<script>
import { ipcRenderer } from "electron";
export default {
  name: "print",
  props: {
    dialogVisible: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      fullPath: "static/print.html", //打印页面
      printList: [], //打印机列表
    };
  },
  mounted() {
    const webview = this.$refs.printWebview;
    // 监听广播  获取打印消息
    webview.addEventListener("ipc-message", (event) => {
      console.log("event :>> ", event.args);
      if (event.channel === "webview-print-do") {
        webview.print(
          {
             //是否是静默打印
            silent: true,
            printBackground: true,
            deviceName: event.args[0].printName,
          },
          (data) => {
             //这个回调是打印后的回调事件，data为true就是打印成功，为false就打印失败
            console.log("data :>> ", data);
          }
        );
      }
    });
  },
  methods: {
    print(printInfo) {
      this.getPrintListHandle(printInfo);
    },
    // 获取打印机列表
    getPrintListHandle(printInfo) {
      // 改用ipc异步方式获取列表，解决打印列数量多的时候导致卡死的问题
      ipcRenderer.send("getPrinterList");
      ipcRenderer.once("getPrinterList", (event, data) => {
        const printList = data.filter((element) => element.status === 0);
        this.printList = printList;
        if (printList.length <= 0) {
          this.$message({
            message: "打印服务异常,请尝试重启电脑",
            type: "error",
          });
        } else {
          this.checkPrinter(printInfo);
        }
      });
    },
    //判断打印机的状态
    checkPrinter(printInfo) {
      console.log("checkPrinter :>> ", printInfo);
      console.log("this.printList :>> ", this.printList);
      const printer = this.printList.find(
        (device) => device.name === printInfo.printName
      );
      console.log("printer :>> ", printer);
      if (printer && printer.status === 0) {
        this.printRender(printInfo);
      } else {
        this.$message({
          message: "当前打印机不可用,请重新设置",
          type: "error",
          duration: 1000,
        });
      }
    },
    //打印
    printRender(printInfo) {
      // 获取<webview>节点
      const webview = this.$refs.printWebview;
      // 发送信息到<webview>里的页面
      // webview.openDevTools();
      webview.send("webview-print-render", {
        printName: printInfo.printName,
        data: printInfo.data,
      });
    },
  },
};
</script>

<style scope lang='scss'>
.container {
  position: fixed;
  left: -500px;
}
</style>