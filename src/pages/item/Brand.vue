<template>
  <v-card>
    <v-card-title>
      <v-btn color="primary"  @click="addBrand">新增品牌</v-btn>
      <!--空间隔离组件-->
      <v-spacer />
      <!--搜索框，与search属性关联-->
      <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
    </v-card-title>

    <v-data-table
      :headers="headers"
      :items="brands"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img v-if="props.item.image" :src="props.item.image" width="130" height="40"/></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="text-xs-center">
          <v-icon small class="mr-2" @click="editItem(props.item)">
            edit
          </v-icon>
          <v-icon small @click="deleteItem(props.item)">
            delete
          </v-icon>
        </td>
      </template>
    </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="500" v-model="show" persistent>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit? '修改品牌':'新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5">
          <BrandForm @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  // 导入自定义的表单组件
  import BrandForm from './BrandForm'

  export default {
    name: "Brand",
    data () {
      return {
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        search: "",
        headers: [ // 头信息
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', value: 'name', sortable: false},
          {text: 'LOGO', align: 'center', value: 'image', sortable: false},
          {text: '首字母', align: 'center', value: 'letter'},
          {text: '操作', align: 'center', value: 'id', sortable: false }
        ],
        show: false,
        oldBrand: {},
        isEdit: false // 是否编辑
      }
    },
    methods: {
      getDataFromServer(){ // 从服务端加载数据的函数
        // 伪造演示数据
        /*const brands = [
          {
            "id": 2032,
            "name": "OPPO",
            "image": "http://img10.360buyimg.com/popshop/jfs/t2119/133/2264148064/4303/b8ab3755/56b2f385N8e4eb051.jpg",
            "letter": "O",
            "categories": null
          },
          {
            "id": 2033,
            "name": "飞利浦（PHILIPS）",
            "image": "http://img12.360buyimg.com/popshop/jfs/t18361/122/1318410299/1870/36fe70c9/5ac43a4dNa44a0ce0.jpg",
            "letter": "F",
            "categories": null
          },
          {
            "id": 2034,
            "name": "华为（HUAWEI）",
            "image": "http://img10.360buyimg.com/popshop/jfs/t5662/36/8888655583/7806/1c629c01/598033b4Nd6055897.jpg",
            "letter": "H",
            "categories": null
          },
          {
            "id": 2036,
            "name": "酷派（Coolpad）",
            "image": "http://img10.360buyimg.com/popshop/jfs/t2521/347/883897149/3732/91c917ec/5670cf96Ncffa2ae6.jpg",
            "letter": "K",
            "categories": null
          },
          {
            "id": 2037,
            "name": "魅族（MEIZU）",
            "image": "http://img13.360buyimg.com/popshop/jfs/t3511/131/31887105/4943/48f83fa9/57fdf4b8N6e95624d.jpg",
            "letter": "M",
            "categories": null
          }
        ];
        // 延迟一段时间，模拟数据请求时间
        setTimeout(()=>{
          this.brands = brands; // 赋值给品牌数组
          this.totalBrands = brands.length; // 赋值数据总条数
          this.loading = false; // 数据加载完成
        }, 1000);*/

       this.loading = true  //加载数据
        this.$http.get("/item/brand/page",{
          params: {
            page: this.pagination.page, // 当前页
            rows: this.pagination.rowsPerPage, // 每页条数
            sortBy: this.pagination.sortBy, // 排序字段
            desc: this.pagination.descending, // 是否降序
            key: this.search // 查询字段
          }
        }).then(resp => { // 获取响应对象
          this.totalBrands = resp.data.total; // 总条数
          this.brands = resp.data.items; // 品牌数据
          this.loading = false; // 加载完成
        })
      },
      addBrand() {
        this.isEdit = false
        this.show = true
        // 把oldBrand变为null
        this.oldBrand = null;
      },
      editItem(oldBrand) {
        // 根据商品信息查询分类进行回显（分类较为特殊）
        this.$http.get("/item/category/bid/" + oldBrand.id)
          .then(({data}) => {
            this.isEdit = true
            // 控制弹窗可见：
            this.show = true;
            // 获取要编辑的brand
            this.oldBrand = oldBrand;
            // 回显商品分类
            this.oldBrand.categories = data;
          })
      },
      closeWindow() {   // 关闭窗口
        this.show = false
      }
    },
    // 渲染后执行
    mounted(){
      this.getDataFromServer()
    },
    watch: {
      pagination: {   // 监视pagination属性的变化，当pagination发生改变时，会调用我们的回调函数，我们在回调函数中进行数据的查询即可！
        handler () {
          this.getDataFromServer()
        },
        deep: true  // deep为true会监视pagination的属性以及属性中对象的变化
      },
      search: {  // 监视search属性
        handler () {
          this.getDataFromServer()
        }
      }
    },
    components: {
      BrandForm
    }
  }
</script>

<!-- scoped:当前样式只作用于当前组件的节点 -->
<style scoped>

</style>
