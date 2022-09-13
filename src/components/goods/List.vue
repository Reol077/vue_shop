<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
            @keyup.enter.native="getGoodsList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格区 -->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="100px"></el-table-column>
        <el-table-column label="商品数量" prop="goods_number" width="100px"></el-table-column>
        <el-table-column label="商品重量(千克)" prop="goods_weight" width="110px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5,10,15,20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      ></el-pagination>
    </el-card>
    <el-dialog title="编辑商品" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRule" ref="editFormRef" label-width="130px">
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格（元）" prop="goods_price">
          <el-input v-model="editForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number">
          <el-input v-model="editForm.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量(千克)" prop="goods_weight">
          <el-input v-model="editForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editShop(editForm.goods_id)">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10
      },
      //   商品列表
      goodslist: [],
      //   总数据条数
      total: 0,
      //   编辑商品对话框
      editDialogVisible: false,
      //   编辑商品数据
      editForm: {},
      //   编辑商品的表单验证
      editFormRule: {
        goods_name: [
          { required: true, message: "商品名称不能为空", trigger: "blur" }
        ],
        goods_price: [
          { required: true, message: "商品价格不能为空", trigger: "blur" }
        ],
        goods_number: [
          { required: true, message: "商品数量不能为空", trigger: "blur" }
        ],
        goods_weight: [
          { required: true, message: "商品重量不能为空", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getGoodsList();
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取商品列表失败");
      //   this.$message.success("获取商品列表成功");
      this.goodslist = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    async removeById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "删除",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      console.log(confirmResult);
      if (confirmResult !== "confirm") {
        return this.$message.info("删除操作已取消!");
      }
      const { data: res } = await this.$http.delete("goods/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败!");
      }
      this.$message.success("删除成功!");
      this.getGoodsList();
    },
    goAddPage() {
      this.$router.push("/goods/add");
    },
    // 编辑商品对话框显示与隐藏
    async showEditDialog(id) {
      const { data: res } = await this.$http.get(`goods/${id}`);
      console.log(res);
      if (res.meta.status !== 200)
        return this.$message.error("获取商品信息失败！");
      this.editForm = res.data;
      console.log(this.editForm);
      this.editDialogVisible = true;
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    editShop(id) {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          `goods/${id}`,
          this.editForm
        );
        if (res.meta.status !== 200)
          return this.$message.error("编辑商品信息失败");
        this.$message.success("编辑商品信息成功");
        this.getGoodsList();
        this.editDialogVisible = false;
      });
    }
  }
};
</script>

<style lang="less" scoped>
</style>