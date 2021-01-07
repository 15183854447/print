<template>
  <div>
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane
        v-for="(item, index) in printList"
        :key="index"
        :label="item.name"
        :name="index.toString()"
        >{{ item.name }}</el-tab-pane
      >
    </el-tabs>
    <Print ref="print"></Print>
  </div>
</template>

<script>
import { ipcRenderer } from "electron";
import path from "path";
import Print from "../component/print";
export default {
  components: {
    Print,
  },
  props: {},
  data() {
    return {
      activeName: 0,
      printList: [], //打印机列表
      fullPath: "static/print.html",
      printName: "",
      dialogVisible: false,
    };
  },
  watch: {},
  computed: {},
  methods: {
    handleClick(tab, event) {
      this.activeName = tab.name;
    },
    //获取打印机列表
    getPrintList() {
      return new Promise((resolve, reject) => {
        //渲染线程主动发送getPrinterList事件到主线程请求打印机列表
        ipcRenderer.send("getPrinterList");
        //监听主线程获取到打印机列表后的回调
        ipcRenderer.once("getPrinterList", (event, data) => {
          //data就是打印机列表
          this.printList = data;
          resolve(data);
        });
      });
    },
  },
  created() {},
  mounted() {
    this.getPrintList(); //加载打印机列表


  //  本地开启服务  接收打印任务
    const Koa = require("koa");
    const app = new Koa();
    const json = require("koa-json");
    const onerror = require("koa-onerror");
    const bodyparser = require("koa-bodyparser");
    const logger = require("koa-logger");
    const router = require("koa-router")();
    // error handler
    onerror(app);
    // middlewares
    app.use(
      bodyparser({
        enableTypes: ["json", "form", "text"],
      })
    );
    app.use(json());
    app.use(logger());

    // logger
    app.use(async (ctx, next) => {
      const start = new Date();
      await next();
      const ms = new Date() - start;
      console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
    });
    // routes
    //获取打印机列表
    router.get("/printList", async (ctx, next) => {
      try {
        await this.getPrintList().then((res) => {
          console.log("res :>> ", res);
          ctx.body = {
            code: 200,
            data: res,
            message: "打印机列表",
          };
        });
      } catch (error) {
        ctx.body = {
          code: 0,
          message: "打印机列表获取失败",
        };
      }
    });
    //打印api
    router.post("/printLabel", async (ctx, next) => {
      console.log("ctx :>> ", ctx.request);
      this.$refs.print.print(ctx.request.body);
      ctx.body = {
        code: 200,
        message: "打印任务创建成功",
      };
      //打印
    });
    // error-handling
    app.on("error", (err, ctx) => {
      console.error("server error", err, ctx);
    });
    app.use(router.routes());
    app.listen(12122, (err) => {
      if (err) {
        console.log("启动失败 :>> ");
      } else {
        console.log("启动成功  端口号：12122");
      }
    });
  },
};
</script>
<style lang="scss" scoped></style>