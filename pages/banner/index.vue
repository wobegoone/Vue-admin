<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ name: '/' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>系统管理</el-breadcrumb-item>
        <el-breadcrumb-item>轮播图管理</el-breadcrumb-item>
        </el-breadcrumb>
        <br>
        <div style="height:62px;">
          <!--搜索框-->
          <el-form :inline="true" style="float:left;" size="small">
              <el-form-item>
                  <nuxt-link :to="{name:'banner-store'}">
                    <el-button type="primary">添加轮播图</el-button>
                  </nuxt-link>
              </el-form-item>
              <el-form-item>
                  <el-input placeholder="搜索轮播图"></el-input>
              </el-form-item>
              <el-form-item>
                  <el-button type="primary">查询</el-button>
              </el-form-item>
          </el-form>
        </div>
        <!--表格数据及操作-->
        <el-table :data="lists" border style="width: 100%" stripe ref="multipleTable" tooltip-effect="dark" @selection-change="selsChange">
            <!--勾选框-->
            <el-table-column type="selection" width="55">
            </el-table-column>
            <!--索引-->
            <el-table-column prop="bannerId" label="ID" width="80">
            </el-table-column>
            <el-table-column prop="name" label="名称">
            </el-table-column>
             <el-table-column prop="status" label="状态" width="120">
                <template slot-scope="scope">
                    <span v-if="scope.row.status === 1">已发布</span>
                    <span v-else>未发布</span>
                </template>
            </el-table-column>
            <el-table-column label="封面" width="170">
                <template slot-scope="scope">
                    <img :src="scope.row.thumb" alt="" style="width:150px;height:50px;">
                </template>
            </el-table-column>
            <el-table-column prop="jumpUrl" label="跳转地址" width="120">
            </el-table-column>
            <el-table-column prop="createdAt" label="创建时间" width="180">
                <template slot-scope="scope">
                    <span>{{scope.row.createdAt | d('yyyy-MM-dd hh:mm:ss')}}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="260">
                <template slot-scope="scope">
                    <nuxt-link :to="{name:'banner-id',params:{ id: scope.row.bannerId }}">
                        <el-button type="info" icon="el-icon-view" size="mini">详情</el-button>
                    </nuxt-link>
                    <nuxt-link :to="{name:'banner-id',params:{ id: scope.row.bannerId }}">
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                    </nuxt-link>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="destroy(scope.row.bannerId,scope.$index, lists)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <br>
        <!--分页条-->
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pageNow"
            :page-sizes="[10, 50, 100, 150]"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="pageCount">
        </el-pagination>
    </div>
</template>


 <script>

  import qs from 'qs'
  import axios from '~/plugins/axios.js'

  export default {
    layout:'frame',
    name:'node',
    async asyncData () {
      const { data } = await axios.get('/admin/banner/lists')
      return {
          pageNow: data.page.now || 1 ,
          pageSize: data.page.size || 10 ,
          pageCount: data.page.count || 0 ,
          lists: data.lists || []
      }
    },
    data() {
      return {
        form: {
          name: '',
          state: true,
          description: ''
        },
        //查询输入框数据
        input: '',
        //导航条默认选项
        activeIndex: '1',
        activeIndex2: '1',
        dialogVisible: false,
        options: [{
          value: '0',
          label: '全部'
        }, {
          value: '1',
          label: '待审核'
        }, {
          value: '2',
          label: '已审核'
        }],
      }
    },
    methods: {
       handleSizeChange(val) {
        axios.get(`/admin/banner/lists?pageSize=${val}`)
        .then(res => {
            this.lists = res.data.lists || [];
            this.pageSize = res.data.page.size || 10;
          });
      },
      handleCurrentChange(val) {
        axios.get(`/admin/banner/lists?pageNow=${val}`)
        .then(res => {
            this.lists = res.data.lists || [];
            this.pageNow = res.data.page.now || 1;
          });
      },
      selsChange(val){
       this.multipleSelection = val;
      },
      destroy(bannerId,index,rows){
        axios.delete(`/admin/bannerId/${bannerId}`, {data: qs.stringify({bannerId:bannerId})})
        .then(res => {
            //判断是否请求成功
            if(res.data.errorId === 'OK'){
                rows.splice(index, 1);
                this.$message({
                    message: '成功删除文章',
                    type: 'success'
                    });    
                }
            }).catch(res => {
                 if(res.response.data.message === ''){
                    this.$message.error('请求异常，请稍后重试！');
                }else{
                    this.$message.error(res.response.data.message);
                }
            });
        }
    }
  };
</script>
