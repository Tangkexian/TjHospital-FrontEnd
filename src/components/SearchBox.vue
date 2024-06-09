<template>
  <el-row>
    <!-- 搜索框部分 -->
    <el-autocomplete
        class="inline-input"
        v-model="state"
        popper-class="my-kqoption"
        :fetch-suggestions="querySearch"
        placeholder="请输入关键词"
        :trigger-on-focus="false"
        @select="handleSelect"
        :popper-append-to-body="false">
    </el-autocomplete>
    &ensp;
    <!-- 搜索按钮部分 -->
    <el-button type="primary" @click="HandleSubmitValue">搜索</el-button>
  </el-row>
</template>

<script>
  export default {
    data() {
      return {
        caretypes: [],
        state: ''
      };
    },
    methods: {
      querySearch(queryString, cb) {
        var caretypes = this.caretypes;
        var results = queryString ? caretypes.filter(this.createFilter(queryString)) : caretypes;
        // 调用 callback 返回建议列表的数据
        cb(results);
      },
      createFilter(queryString) {
        return (restaurant) => {
          return (restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
        };
      },
      handleSelect(item) {
        console.log(item);
      },
      //提交按钮的事件，把子组件的所有数据提交给父组件
      HandleSubmitValue() {
        this.$emit("SearchBoxValueToParent", this.state);
      },
    },
    mounted() {
      this.caretypes = this.input_data;
    },
    props:{
      input_data:{
        type: Array,
        required: true,
      },
      input_width: {
        type: String,
        default: '100%',
      }
    }
  }
</script>

<style lang="less" scoped>
& /deep/ .my-kqoption {
    width: 200%!important; 
}
</style>

<style scoped>
.inline-input {
  width: input_width;
}
</style>