<template>
  <v-card>
    <v-toolbar class="elevation-0">
      <v-btn
        color="blue-grey darken-1"
        @click="addGoods"
        style="color: white"
        small
        >Añadir Producto</v-btn
      >
      <v-spacer />
      <v-flex xs3>
        Estado：
        <v-btn-toggle mandatory v-model.lazy="filter.saleable">
          <v-btn flat small color="blue-grey darken-1"> Todo </v-btn>
          <v-btn flat :value="true" small color="blue-grey darken-1">
            Alta
          </v-btn>
          <v-btn flat :value="false" small color="blue-grey darken-1">
            Baja
          </v-btn>
        </v-btn-toggle>
      </v-flex>
      <v-flex xs3>
        <v-text-field
          append-icon="search"
          label="Búsqueda"
          single-line
          hide-details
          v-model="filter.search"
        />
      </v-flex>
    </v-toolbar>
    <v-divider />
    <v-data-table
      :headers="headers"
      :items="goodsList"
      :pagination.sync="pagination"
      :total-items="totalGoods"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.title }}</td>
        <td class="text-xs-center">{{ props.item.cname }}</td>
        <td class="text-xs-center">{{ props.item.bname }}</td>
        <td class="justify-center layout px-0">
          <v-btn color="primary" icon @click="editGoods(props.item)">
            <i class="el-icon-edit" />
          </v-btn>
          <v-btn color="warning" icon @click="deleteGoods(props.item.id)">
            <i class="el-icon-delete" />
          </v-btn>
          <v-btn
            icon
            v-if="props.item.saleable"
            color="pink lighten-2"
            style="color: white"
            @click="changeSaleable(props.item.id)"
            >BAJA</v-btn
          >
          <v-btn
            icon
            v-else
            color="success"
            @click="changeSaleable(props.item.id)"
            >ALTA</v-btn
          >
        </td>
      </template>
    </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="800" v-model="show" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="blue-grey darken-1">
          <v-toolbar-title
            >{{ isEdit ? "Editar " : "Añadir " }}Producto</v-toolbar-title
          >
          <v-spacer />
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-3" style="height: 600px">
          <goods-form
            :oldGoods="oldGoods"
            :step="step"
            @close="closeWindow"
            :is-edit="isEdit"
            ref="goodsForm"
          />
        </v-card-text>
        <!--底部按钮，用来操作步骤线-->
        <v-card-actions class="elevation-10">
          <v-flex class="xs3 mx-auto">
            <v-btn
              @click="previous"
              small
              color="blue-grey darken-1"
              :disabled="step === 1"
              style="color: white"
              >Anterior</v-btn
            >
            <v-btn
              @click="next"
              small
              color="blue-grey darken-1"
              :disabled="step === 4"
              style="color: white"
              >Siguiente</v-btn
            >
          </v-flex>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
// 导入自定义的表单组件
import GoodsForm from "./GoodsForm";

export default {
  name: "goods",
  data() {
    return {
      headers: [
        { text: "ID", align: "center", value: "id" },
        { text: "TÍTULO", align: "center", sortable: false, value: "title" },
        { text: "CATEGORÍA", align: "center", sortable: false, value: "cname" },
        { text: "MARCA", align: "center", value: "bname", sortable: false },
        { text: "OP.", align: "center", sortable: false },
      ],
      filter: {
        saleable: true, // 上架还是下架
        search: "", // 搜索过滤字段
      },
      totalGoods: 0, // 总条数
      goodsList: [], // 当前页品牌数据
      loading: true, // 是否在加载中
      pagination: {}, // 分页信息
      show: false, // 控制对话框的显示
      oldGoods: {}, // 即将被编辑的商品信息
      isEdit: false, // 是否是编辑
      step: 1, // 子组件中的步骤线索引，默认为1
    };
  }, //与监听重复
  /* mounted() { // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },*/ watch: {
    pagination: {
      // 监视pagination属性的变化
      deep: true, // deep为true，会监视pagination的属性及属性中的对象属性变化
      handler() {
        // 变化后的回调函数，这里我们再次调用getDataFromServer即可
        this.getDataFromServer();
      },
    },
    filter: {
      // 监视搜索字段
      handler() {
        this.getDataFromServer();
      },
      deep: true,
    },
  },
  methods: {
    getDataFromServer() {
      // 从服务的加载数的方法。
      // 发起请求
      this.$http
        .get("/item/spu/page", {
          params: {
            key: this.filter.search, // 搜索条件
            saleable: this.filter.saleable === 0 ? null : this.filter.saleable, // 上下架
            page: this.pagination.page, // 当前页
            rows:
              this.pagination.rowsPerPage < 0
                ? this.totalGoods
                : this.pagination.rowsPerPage, // 每页大小
            sortBy: this.pagination.sortBy, // 排序字段
            desc: this.pagination.descending, // 是否降序
          },
        })
        .then((resp) => {
          // 这里使用箭头函数
          this.goodsList = resp.data.items;
          this.totalGoods = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        });
    },
    addGoods() {
      // 修改标记
      this.isEdit = false;
      // 控制弹窗可见：
      this.show = true;
      // 把oldBrand变为null
      this.oldGoods = {};
    },
    async editGoods(oldGoods) {
      // 发起请求，查询商品详情和skus
      oldGoods.spuDetail = await this.$http.loadData(
        "/item/spu/detail/" + oldGoods.id
      );
      oldGoods.skus = await this.$http.loadData(
        "/item/sku/list?id=" + oldGoods.id
      );
      // 修改标记
      this.isEdit = true;
      // 控制弹窗可见：
      this.show = true;
      // 获取要编辑的goods
      this.oldGoods = oldGoods;
    },
    deleteGoods(id) {
      this.$message
        .confirm("¿Se va a eliminar de forma permanente, continua?")
        .then(() => {
          this.$http
            .delete("/item/goods/spu/" + id)
            .then(() => {
              this.getDataFromServer();
              this.$message.info("¡Eliminado!");
            })
            .catch(() => {
              this.$message.error("¡Error!");
            });
        })
        .catch(() => {
          this.$message.info("¡Cancelado!");
        });
    },
    changeSaleable(id) {
      this.$http
        .put("item/goods/spu/out/" + id)
        .then(() => {
          this.$message.success("¡Éxito!");
          this.getDataFromServer();
        })
        .catch(() => {
          this.$message.error("¡Error!");
        });
    },
    closeWindow() {
      console.log(1);
      // 重新加载数据
      this.getDataFromServer();
      // 关闭窗口
      this.show = false;
      // 将步骤调整到1
      this.step = 1;
    },
    previous() {
      if (this.step > 1) {
        this.step--;
      }
    },
    next() {
      if (this.step < 4 && this.$refs.goodsForm.$refs.basic.validate()) {
        this.step++;
      }
    },
  },
  components: {
    GoodsForm,
  },
};
</script>

<style scoped>
</style>
