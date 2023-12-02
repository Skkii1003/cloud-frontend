<template>
  <div class="container">
    <h1>案件检索</h1>

    <!-- 搜索框 -->
    <div class="search">
      <el-input clearable v-model="keyword">
        <template #prepend>
          <el-select v-model="searchWay" placeholder="Select" style="width: 115px">
            <el-option label="全类型" value="all" />
            <el-option label="案件事实" value="fact" />
            <el-option label="案件原因" value="reason" />
            <el-option label="案件结果" value="result" />
          </el-select>
        </template>
        <template #append>
          <el-button :icon="Search" @click="search" @keyup.enter="search"></el-button>
        </template>
      </el-input>
    </div>
    <!-- 案件列表 -->
    <div class="list">
      <el-table :data="case_list" table-layout="fixed" height="80%"  show-overflow-tooltip stripe>
        <el-table-column prop="pid" min-width="10%" label="案件编号"  />
        <el-table-column prop="fact" min-width="15%" label="案件事实"  />
        <el-table-column prop="reason"  min-width="15%" label="案件原因"  />
        <el-table-column prop="result" min-width="15%" label="案件结果"  />
        <el-table-column prop="charge" min-width="15%" label="罪名"  />
        <el-table-column prop="article" min-width="15%" label="法条"  />
        <el-table-column fixed="right" min-width="15%">
          <template #default="scope">
            <el-button link type="primary" size="small" @click="toCaseInfo(scope.row.pid)">查看详情</el-button>
            <el-button link type="primary" size="small" @click="toSimilarCase(scope.row.pid)">相似案件</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 分页 -->
    <div class="page">
      <el-pagination background layout="prev , next" prev-text="上一页" next-text="下一页" :page-size="10" :current-page="page"
        :total="100" @current-change="pagination" />
    </div>

    <!-- 案件详情弹窗 -->
    <el-dialog v-model="showCaseInfo" title="案件详情"  :destroy-on-close="true" :align-center="true">
      <el-descriptions :column="1" :border="false" size="large">
        <el-descriptions-item label="案件编号" label-class-name="label">{{caseInfo.pid}}</el-descriptions-item>
        <el-descriptions-item label="全文" label-class-name="label">{{caseInfo.qw}}</el-descriptions-item>
        <el-descriptions-item label="案件事实" label-class-name="label">{{caseInfo.fact}}</el-descriptions-item>
        <el-descriptions-item label="案件原因" label-class-name="label">{{caseInfo.reason}}</el-descriptions-item>
        <el-descriptions-item label="案件结果" label-class-name="label">{{caseInfo.result}}</el-descriptions-item>
        <el-descriptions-item label="罪名" label-class-name="label">
          <el-tag v-for="item in caseInfo.charge" :key="item">{{item}}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="法条" label-class-name="label">
          <el-tag v-for="item in caseInfo.article" :key="item">{{item}}</el-tag>
        </el-descriptions-item>
      </el-descriptions>
    </el-dialog>
    
    <!-- 相似案件弹窗 -->
    <el-dialog v-model="showSimilarCase" title="相似案件" :destroy-on-close="true" :align-center="true">
      <el-table :data="similarCase" table-layout="fixed" show-overflow-tooltip stripe>
        <el-table-column prop="pid" label="案件编号"  />
        <el-table-column prop="fact" label="案件事实"  />
        <el-table-column prop="reason" label="案件原因"  />
        <el-table-column prop="result" label="案件结果"  />
        <el-table-column prop="charge" label="罪名"  />
        <el-table-column prop="article" label="法条"  />          
      </el-table>
      
    </el-dialog>

  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Search } from '@element-plus/icons-vue';
import axios from 'axios';
import { ElMessage } from 'element-plus';

//检索关键词数据
let keyword = ref('');
//检索按钮回调
const search = () => {
  axios.post('http://localhost:8090/candidate/search', {
    keyword: keyword.value,
    searchWay: searchWay.value,
    page: page.value,
    size: 10,
  }).then((res) => {
    case_list.value = res.data.obj;
  }).catch((err) => {
    console.log(err);
    ElMessage.error('检索失败');
  })
};
//检索类型数据
let searchWay = ref('all');
//案件列表数据
let case_list = ref([{}])
case_list.value = [{
  pid: 1,
  fact: 'this is fact',
  reason: 'this is reason',
  result: 'this is result',
  charge: ['this is charge1', 'this is charge2'],
  article: ['this is article1', 'this is article2'],
},
{
  pid: 2,
  fact: 'this is fact',
  reason: 'this is reason',
  result: 'this is result',
  charge: ['this is charge1', 'this is charge2'],
  article: ['this is article1', 'this is article2'],
},
{
  pid: 3,
  fact: 'this is fact',
  reason: 'this is reason',
  result: 'this is result',
  charge: ['this is charge1', 'this is charge2'],
  article: ['this is article1', 'this is article2'],
},
]
//分页数据
let page = ref(1);
//分页回调
const pagination = (val: number) => {
  page.value = val;
  search();
}

//案件详情弹窗
let showCaseInfo = ref(false);
let caseInfo = ref({})
const toCaseInfo = (id: number) => {
  axios.get('http://localhost:8090/candidate/detail/'+id.toString()).then((res) => {
    caseInfo.value = res.data.obj;
    showCaseInfo.value = true;
  }).catch((err) => {
    console.log(err);
    ElMessage.error('获取案件详情失败');
  })

  //测试数据
  caseInfo.value = {
    pid: 1,
    qw: 'this is qw',
    fact: 'this is fact',
    reason: 'this is reason',
    result: 'this is result',
    charge: ['this is charge1', 'this is charge2'],
    article: ['this is article1', 'this is article2'],
  }
  showCaseInfo.value = true;
}

//相似案件弹窗
let showSimilarCase = ref(false);
let similarCase = ref([{}])
const toSimilarCase = (id: number) => {
  axios.post('http://localhost:8090/candidate/similar/'+id.toString()).then((res) => {
    similarCase.value = res.data.obj;
    showSimilarCase.value = true;
  }).catch((err) => {
    console.log(err);
    ElMessage.error('获取相似案件失败');
  })

  //测试数据
  similarCase.value = case_list.value;
  showSimilarCase.value = true;
}


</script>

<style lang="scss">
.container {
  width: 100%;
  height: 100vh;

  h1 {
    text-align: center;
    font-size: 25px;
  }
}

.search {
  width: 60%;
  justify-content: center;
  margin: 20px auto;
}

.list {
  width: 60%;
  margin: 0 auto;
}

.el-pagination {
  margin: 20px auto;
  justify-content: center;
}
.label{
  font-size: 18px;
}
</style>