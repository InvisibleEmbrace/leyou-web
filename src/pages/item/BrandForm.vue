<template>
  <v-form v-model="valid">
    <v-text-field v-model="brand.name" label="请输入品牌名称" required/>
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required/>
    <v-cascader
      url="/item/category/list"
      multiple
      required
      v-model="brand.categories"
      label="请选择商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image"
          url="/upload"
          :multiple="false"
          :pic-width="250"
          :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear" ref="BrandForm">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  import VCascader from '../../components/cascader/Cascader'
  import VUpload from '../../components/form/Upload'
  export default {
    name: "BrandForm.vue",
    data() {
      return {
        valid: false, // 表单校验结果标记
        brand: {
          name: '', // 品牌名称
          letter: '', // 品牌首字母
          image: '',// 品牌logo
          categories: [], // 品牌所属的商品分类数组
        }
      }
    },
    methods: {
      submit(){
        // 提交表单
      },
      clear(){
        // 重置表单
        this.$refs.BrandForm.reset();
        // 需要手动清空商品分类
        this.categories = [];
      }
    },
    components: {
      VCascader,
      VUpload
    }
  }
</script>

<style scoped>

</style>
