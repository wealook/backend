<style lang="less"></style>
<template>
  <div class="">
    <div class="table-basic-vue frame-page h-panel">
      <div class="h-panel-bar"><span class="h-panel-title">编辑角色</span></div>
      <div class="h-panel-body">
        <p>
          <Button color="blue" icon="icon-arrow-left" @click="back()">返回列表</Button>
        </p>

        <Form v-width="400" ref="form" :validOnChange="true" :showErrorTip="true" :labelWidth="110" :rules="rules" :model="role">
          <FormItem label="角色名" prop="display_name">
            <template v-slot:label>角色名</template>
            <input type="text" v-model="role.display_name" />
          </FormItem>
          <FormItem label="描述" prop="description">
            <template v-slot:label>描述</template>
            <input type="text" v-model="role.description" />
          </FormItem>
          <FormItem>
            <Button color="primary" @click="create">保存</Button>
          </FormItem>
        </Form>
      </div>
    </div>
  </div>
</template>
<script>
import Role from 'model/AdministratorRole';

export default {
  props: ['id'],
  data() {
    return {
      role: Role.parse({}),
      rules: {
        required: ['display_name', 'description']
      }
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      R.AdministratorRole.Edit({id:this.id}).then(resp => {
        this.role = resp.data;
      });
    },
    back() {
      this.$router.push({ name: 'AdministratorRole' });
    },
    create() {
      let validResult = this.$refs.form.valid();
      if (validResult.result) {
        R.AdministratorRole.Update(this.role).then(resp => {
          HeyUI.$Message.success('成功');
          this.$router.push({ name: 'AdministratorRole' });
        });
      }
    }
  }
};
</script>
