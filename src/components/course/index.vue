<template>
  <div class="table-basic-vue frame-page h-panel">
    <div class="h-panel-bar"><span class="h-panel-title">课程</span></div>
    <div class="h-panel-bar">
      <Form :labelWidth="110">
        <FormItem label="关键字搜索" prop="avatar">
          <input type="text" v-model="pagination.keywords" placeholder="课程标题" />
        </FormItem>
        <FormItem>
          <Button color="primary" @click="getData(true)">搜索</Button>
        </FormItem>
      </Form>
    </div>
    <div class="h-panel-body">
      <p>
        <Button color="blue" icon="h-icon-plus" @click="create()">添加</Button>
      </p>
      <Table :loading="loading" :datas="datas">
        <TableItem :width="70" prop="id" title="ID"></TableItem>
        <TableItem title="封面">
          <template slot-scope="{ data }">
            <Avatar type="female" :src="data.thumb" noInfo></Avatar>
          </template>
        </TableItem>
        <TableItem prop="title" title="课程"></TableItem>
        <TableItem prop="charge" title="价格" unit="元"></TableItem>
        <TableItem prop="published_at" title="上线时间"></TableItem>
        <TableItem title="显示">
          <template slot-scope="{ data }">
            <span v-if="data.is_show === 1">是</span>
            <span v-else>否</span>
          </template>
        </TableItem>
        <TableItem title="操作" align="center" :width="80">
          <template slot-scope="{ data }">
            <Poptip content="确认删除？" @confirm="remove(datas, data)">
              <button class="h-btn h-btn-s h-btn-red">删除</button>
            </Poptip>
            <button class="h-btn h-btn-s h-btn-primary" @click="edit(data)">编辑</button>
            <button class="h-btn h-btn-s" @click="goChapter(data)">章节</button>
          </template>
        </TableItem>
      </Table>
      <p></p>
      <Pagination v-if="pagination.total > 0" align="right" v-model="pagination" @change="changePage" />
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      pagination: {
        page: 1,
        size: 20,
        total: 0,
        keywords: '',
      },
      datas: [],
      loading: false
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.getData(true);
    },
    changePage() {
      this.getData();
    },
    getData(reload = false) {
      if (reload) {
        this.pagination.page = 1;
      }
      this.loading = true;
      R.Course.List(this.pagination).then(resp => {
        this.datas = resp.data.data;
        this.pagination.total = resp.data.total;
        this.pagination.page = resp.data.current_page;
        this.pagination.size = resp.data.per_page;
        this.loading = false;
      });
    },
    create() {
      this.$router.push({ name: 'CourseCreate' });
    },
    remove(data, item) {
      R.Course.Delete({ id: item.id }).then(resp => {
        HeyUI.$Message.success('成功');
        this.getData(true);
      });
    },
    edit(item) {
      this.$router.push({ name: 'CourseEdit', params: { id: item.id } });
    },
    goChapter(item) {
      this.$router.push({ name: 'CourseChapter', params: { cid: item.id } });
    }
  }
};
</script>
