<template>
  <div>
    <el-dialog
      :title="!dataForm.id ? '新增' : '修改'"
      :close-on-click-modal="false"
      :visible.sync="visible"
    >
      <el-form
        :model="dataForm"
        :rules="dataRules"
        ref="dataForm"
        @keyup.enter.native="dataFormSubmit()"
        label-width="120px"
      >
        <el-form-item label="昵称" prop="nickname">
          <el-input v-model="dataForm.nickname" placeholder="昵称"></el-input>
        </el-form-item>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="dataForm.username" placeholder="用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input
            type="password"
            v-model="dataForm.password"
            placeholder="密码"
          ></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="passwordCheck">
          <el-input
            type="password"
            v-model="dataForm.passwordCheck"
            placeholder="确认密码"
          ></el-input>
        </el-form-item>
        <el-form-item label="头像" prop="header">
          <single-upload v-model="dataForm.header"></single-upload>
        </el-form-item>
        <el-form-item label="手机号码" prop="mobile">
          <el-input v-model="dataForm.mobile" placeholder="手机号码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
        </el-form-item>
        <el-form-item prop="levelId" label="会员等级">
          <el-select v-model="dataForm.levelId" placeholder="请选择会员等级">
            <el-option
              v-for="item in memberLevelList"
              :key="item.levelId"
              :label="item.name"
              :value="item.levelId"
            ></el-option></el-select
        ></el-form-item>
        <el-form-item label="性别" prop="gender">
          <el-select v-model="dataForm.gender" placeholder="请选择性别">
            <el-option :value="0" label="男"></el-option>
            <el-option :value="1" label="女"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="生日" prop="birth">
          <el-date-picker
            v-model="dataForm.birth"
            type="date"
            placeholder="选择出生日期"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="所在城市" prop="city">
          <el-cascader
            :options="provinceAndCityData"
            v-model="dataForm.city"
            placeholder="请选择所在城市"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="职业" prop="job">
          <el-input v-model="dataForm.job" placeholder="职业"></el-input>
        </el-form-item>
        <el-form-item label="个性签名" prop="sign">
          <el-input v-model="dataForm.sign" placeholder="个性签名"></el-input>
        </el-form-item>
        <el-form-item v-if="dataForm.id" label="积分" prop="integration">
          <el-input
            v-model="dataForm.integration"
            placeholder="积分"
          ></el-input>
        </el-form-item>
        <el-form-item v-if="dataForm.id" label="成长值" prop="growth">
          <el-input v-model="dataForm.growth" placeholder="成长值"></el-input>
        </el-form-item>
        <el-form-item label="启用状态" prop="status">
          <el-switch
            v-model="dataForm.status"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch> </el-form-item
      ></el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》'
import { provinceAndCityData } from "element-china-area-data";
import SingleUpload from "@/components/upload/singleUpload.vue";

export default {
  //import引入的组件需要注入到对象中才能使用
  components: { SingleUpload },
  props: {},
  data() {
    //这里存放数据
    return {
      provinceAndCityData: provinceAndCityData,
      visible: false,
      dataForm: {
        integration: 0,
        growth: 0,
        status: 1,
        password: "",
        passwordCheck: "",
        username: "",
        nickname: "",
        mobile: "",
        email: "",
        job: "",
        sign: "",
        birth: "",
        city: [],
        gender: "",
        header: "",
      },
      dataRules: {
        levelId: [
          { required: true, message: "会员等级id不能为空", trigger: "blur" },
        ],
        nickname: [
          { required: true, message: "昵称不能为空", trigger: "blur" },
        ],
        username: [
          { required: true, message: "用户名不能为空", trigger: "blur" },
        ],
        password: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (value === "") {
                callback(new Error("密码不能为空"));
              } else {
                if (this.dataForm.passwordCheck !== "") {
                  this.$refs.dataForm.validateField("passwordCheck");
                }
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        passwordCheck: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (value === "") {
                callback(new Error("请确认密码"));
              } else if (value !== this.dataForm.password) {
                callback(new Error("两次输入密码不一致，请重新输入"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (!value) {
                callback(new Error("请输入手机号"));
              } else if (!/^1(3|4|5|6|7|8)\d{9}$/.test(value)) {
                callback(new Error("请输入正确的十一位手机号码"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        email: [
          {
            required: true,
            message: "请输入正确的邮箱地址",
            trigger: "blur",
          },
        ],
        gender: [{ required: true, message: "请选择性别", trigger: "blur" }],
        birth: [{ required: true, message: "请选择生日", trigger: "blur" }],
        city: [{ required: true, message: "请选择所在城市", trigger: "blur" }],
      },
      memberLevelList: [],
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    getMemberLevelList() {
      this.$http({
        url: this.$http.adornUrl("/member/memberlevel/getMemberLevelList"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.memberLevelList = data.resultList;
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    dataFormSubmit() {
      let postData = {};
      Object.assign(postData, this.dataForm, {
        city: this.dataForm.city.join(","),
      });
      this.$http({
        url: this.$http.adornUrl(
          `/member/member/${this.dataForm.id == 0 ? "save" : "update"}`
        ),
        method: "post",
        data: this.$http.adornData(postData, false),
      }).then(({ data }) => {
        if (data.code === 0) {
          this.$message({
            type: "success",
            message: "操作成功",
            duration: 1000,
            onClose: () => {
              this.visible = false;
              this.$emit("refreshDataList");
            },
          });
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/member/member/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams({}),
          }).then(({ data }) => {
            if (data.code === 0) {
              this.dataForm = data.member;
              this.dataForm.city = data.member.city.split(",");
              this.dataForm.passwordCheck = data.member.password;
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMemberLevelList();
  },
  //声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>