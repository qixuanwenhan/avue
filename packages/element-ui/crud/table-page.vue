<template>
  <el-pagination :class="b('pagination')"
                 v-show="pageFlag"
                 :small="crud.isMobile"
                 :hide-on-single-page="vaildData(crud.tableOption.simplePage,config.simplePage)"
                 :pager-count="defaultPage.pagerCount"
                 :current-page.sync="defaultPage.currentPage"
                 :background="vaildData(defaultPage.pageBackground,config.pageBackground)"
                 :page-size="defaultPage.pageSize"
                 :page-sizes="defaultPage.pageSizes"
                 @size-change="sizeChange"
                 @current-change="currentChange"
                 layout="total, sizes, prev, pager, next, jumper"
                 :total="defaultPage.total"></el-pagination>
</template>

<script>
import config from "./config";
import create from "core/create";
export default create({
  name: "crud",
  inject: ["crud"],
  props: {
    page: {
      type: Object,
      default: () => {
        return {}
      }
    }
  },
  data () {
    return {
      config: config,
      defaultPage: {
        total: 0, // 总页数
        pagerCount: 7,//超过多少条隐藏
        currentPage: 1, // 当前页数
        pageSize: 10, // 每页显示多少条
        pageSizes: [10, 20, 30, 40, 50, 100],
        background: true // 背景颜色
      }
    };
  },
  created () {
    this.pageInit();
    this.crud.$emit("on-load", this.defaultPage);
  },
  watch: {
    page: {
      handler () {
        this.pageInit();
      },
      deep: true,
    },
    //如果当前页面删除没数据了调用第一页
    'defaultPage.total' (val) {
      if (this.defaultPage.total === (this.defaultPage.currentPage - 1) * this.defaultPage.pageSize && this.defaultPage.total != 0) {
        this.defaultPage.currentPage = this.defaultPage.currentPage - 1;
        this.crud.$emit("on-load", this.defaultPage);
        this.crud.$emit("current-change", this.defaultPage.currentPage);
        this.updateValue();
      }
    }
  },
  computed: {
    pageFlag () {
      return this.defaultPage.total != 0
    }
  },
  methods: {
    pageInit () {
      this.defaultPage = Object.assign(this.defaultPage, this.page)
      this.updateValue();
    },
    updateValue () {
      this.crud.$emit('update:page', this.defaultPage)
    },
    // 页大小回调
    sizeChange (val) {
      this.defaultPage.currentPage = 1;
      this.defaultPage.pageSize = val;
      this.updateValue();
      this.crud.$emit("on-load", this.defaultPage);
      this.crud.$emit("size-change", val);
    },
    // 页码回调
    currentChange (val) {
      this.updateValue();
      this.crud.$emit("on-load", this.defaultPage);
      this.crud.$emit("current-change", val);
    }
  }
});
</script>