<style lang="less"></style>
<template>
  <div class="">
    <div class="table-basic-vue frame-page h-panel">
      <div class="h-panel-bar"><span class="h-panel-title">编辑视频</span></div>
      <div class="h-panel-body">
        <p>
          <Button color="blue" icon="icon-arrow-left" @click="back()">返回列表</Button>
        </p>

        <Form ref="form" :validOnChange="true" :showErrorTip="true" :labelWidth="110" :rules="rules" :model="video">
          <FormItem label="所属课程" prop="course_id">
            <template v-slot:label>所属课程</template>
            <Select v-model="video.course_id" :datas="courses" keyName="id" titleName="title" :filterable="true" @change="selectCourse"></Select>
          </FormItem>
          <FormItem label="章节" prop="chapter_id">
            <template v-slot:label>章节</template>
            <Select v-model="video.chapter_id" :datas="chapters" keyName="id" titleName="title" :filterable="true"></Select>
          </FormItem>
          <FormItem label="视频名" prop="title">
            <template v-slot:label>视频名</template>
            <input type="text" v-model="video.title" />
          </FormItem>
          <FormItem label="价格" prop="charge">
            <template v-slot:label>价格</template>
            <div class="h-input-group" v-width="200">
              <input type="text" v-model="video.charge" />
              <span class="h-input-addon">元</span>
            </div>
          </FormItem>
          <FormItem label="简短介绍" prop="short_description">
            <template v-slot:label>简短介绍</template>
            <textarea v-model="video.short_description"></textarea>
          </FormItem>
          <FormItem label="详细介绍" prop="description">
            <template v-slot:label>详细介绍</template>
            <div ref="editor"></div>
          </FormItem>
          <FormItem label="上架时间" prop="published_at">
            <template v-slot:label>上架时间</template>
            <DatePicker v-model="video.published_at" v-width="200" type="datetime"></DatePicker>
          </FormItem>
          <FormItem label="是否显示" prop="is_show">
            <template v-slot:label>是否显示</template>
            <h-switch v-model="video.is_show" :trueValue="1" :falseValue="-1"></h-switch>
          </FormItem>
          <FormItem label="SEO描述" prop="seo_description">
            <template v-slot:label>SEO描述</template>
            <textarea v-model="video.seo_description"></textarea>
          </FormItem>
          <FormItem label="SEO关键字" prop="seo_keywords">
            <template v-slot:label>SEO关键字</template>
            <textarea v-model="video.seo_keywords"></textarea>
          </FormItem>

          <FormItem label="上传视频">
            <template v-slot:label>上传视频</template>
            <Tabs :datas="tabs" v-model="tab"></Tabs>
          </FormItem>

          <FormItem label="阿里云视频ID" prop="aliyun_video_id" v-show="tab === '阿里云'">
            <template v-slot:label>阿里云视频ID</template>
            <div class="h-input-group">
              <input type="text" v-model="video.aliyun_video_id" />
              <Button @click="openAliyunWindow" color="yellow">点这里上传视频到阿里云获取ID</Button>
            </div>
          </FormItem>
          <FormItem label="腾讯云视频ID" prop="tencent_video_id" v-show="tab === '腾讯云'">
            <template v-slot:label>腾讯云视频ID</template>
            <div class="h-input-group">
              <input type="text" v-model="video.tencent_video_id" />
              <Button @click="openTencentWindow" color="primary">点这里上传视频到腾讯云获取ID</Button>
            </div>
          </FormItem>
          <FormItem label="直链URL" prop="url" v-show="tab === '直链'">
            <template v-slot:label>直链URL</template>
            <input type="text" v-model="video.url" />
          </FormItem>

          <FormItem label="视频时长" prop="duration">
            <template v-slot:label>视频时长</template>
            <div class="h-input-group" v-width="200">
              <input type="text" v-model="video.duration" />
              <span class="h-input-addon">秒</span>
            </div>
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
import WangEditor from 'wangeditor';
import '../../css/richtext-editor.less';

import Video from 'model/Video';

export default {
  props: ['id'],
  data() {
    return {
      video: Video.parse({}),
      courses: [],
      chapters: [],
      tabs: ['阿里云', '腾讯云', '直链'],
      tab: '阿里云',
      rules: {
        required: ['course_id', 'title', 'charge', 'short_description', 'published_at', 'is_show', 'seo_description', 'seo_keywords']
      }
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    tabActive() {
      if (this.video.aliyun_video_id) {
        this.tab = '阿里云';
      }
      if (this.video.tencent_video_id) {
        this.tab = '腾讯云';
      }
      if (this.video.url) {
        this.tab = '直链';
      }
    },
    init() {
      this.video.id = this.id;
      this.video.is_show = 0;

      var editor = new WangEditor(this.$refs.editor);
      editor.customConfig.onchange = html => {
        this.video.description = html;
      };
      editor.customConfig.uploadImgServer = '/backend/api/v1/upload/image';
      editor.customConfig.uploadImgMaxLength = 1;
      editor.customConfig.uploadFileName = 'file';
      editor.customConfig.uploadImgHeaders = {
        Authorization: 'Bearer ' + Utils.getLocal('token')
      };
      editor.create();

      // 读取创建所需要的参数
      R.Video.CreateParams().then(resp => {
        this.courses = resp.data.courses;
      });

      R.Video.Edit({ id: this.id }).then(resp => {
        this.video = resp.data.video;
        editor.txt.html(this.video.description);
        this.tabActive();
        this.selectCourse({ id: this.video.course_id });
      });
    },
    back() {
      this.$router.push({ name: 'Video' });
    },
    create() {
      let validResult = this.$refs.form.valid();
      if (validResult.result) {
        R.Video.Update(this.video).then(resp => {
          HeyUI.$Message.success('添加成功');
          this.$router.push({ name: 'Video' });
        });
      }
    },
    selectCourse(course) {
      R.CourseChapter.List({ course_id: course.id }).then(resp => {
        this.chapters = resp.data.chapters;
      });
    },
    openAliyunWindow() {
      window.open(this.getCurrentDomain() + '/backend/video/upload/aliyun?token=' + Utils.getLocal('token'));
    },
    openTencentWindow() {
      window.open(this.getCurrentDomain() + '/backend/video/upload/tencent?token=' + Utils.getLocal('token'));
    },
    getCurrentDomain() {
      return window.location.protocol + '//' + window.location.host;
    }
  }
};
</script>
