<template>
  <div class="cash">
    <van-nav-bar title="返利提现" left-text="返回" left-arrow @click-left="$router.push('/user')"/>
    <div style="margin-top:10px">
      <van-cell-group title="提现">
        <van-cell>
          <van-field :value="hasTaobi" label="返利总额(元)" left-icon="gold-coin" disabled/>
        </van-cell>

        <van-cell>
          <van-field
            type="number"
            v-model="taobi"
            required
            clearable
            label="提现金额(元)"
            left-icon="alipay"
            placeholder="请输入提现金额"
          />
        </van-cell>
      </van-cell-group>
      <van-cell-group>
        <van-cell :center="true">
          <van-button @click="postCash" size="large" type="info">确认提交</van-button>
        </van-cell>
      </van-cell-group>
      <notice notice="请务必确认提现支付宝账号正确，提现金额必须大于5元"></notice>
    </div>
  </div>
</template>

<script>
import { Icon, Cell, CellGroup, Field, NavBar, Button } from "vant";
import Notice from "@/components/Notice";

export default {
  components: {
    [NavBar.name]: NavBar,
    [Icon.name]: Icon,
    [Cell.name]: Cell,
    [CellGroup.name]: CellGroup,
    [Field.name]: Field,
    [Button.name]: Button,
    Notice
  },
  data() {
    return {
      hasTaobi: 0,
      taobi: "",
      aliCount: false
    };
  },
  mounted() {
    this.checkAliCount();
  },
  activated() {
    thsi.checkAliCount();
  },
  computed: {},
  methods: {
    checkAliCount() {
      let url = `/tixian`;
      this.axios.get(url).then(res => {
        let result = res.data;
        this.aliCount = result.data.zhifubao ? true : false;
        this.hasTaobi = result.data.taobi;
        if (!this.aliCount) {
          this.$dialog({
            title: "请完善资料",
            message: "请填写真实支付宝账号和真实姓名，用于提现使用。"
          }).then(() => {
            this.$router.push("/alipay");
          });
        }
      });
    },
    postCash() {
      if (!this.aliCount) {
        this.$toast("请先完善个人资料，填写提现");
        return false;
      }
      if (!this.taobi) {
        this.$toast("请输入提现金额");
        return false;
      }
      let _number = Number(this.taobi);
      if (!_number) {
        this.$toast("请输入正确的提现金额");
        return false;
      }
      if (_number < 5 || _number > 10000) {
        this.$toast("请输入正确的提现金额 ");
        return false;
      }
      if (_number > this.hasTaobi) {
        this.$toast("申请提现金额大于可提现额度");
        return false;
      }
      let url = "/tixian";
      let form = new FormData();
      form.append("taobi", this.taobi);
      this.axios.post(url, form).then(res => {
        let result = res.data;
        if (result.code == 200) {
          this.$toast("提交成功");
          this.$router.back();
        } else {
          this.$toast(result.msg);
        }
      });
    }
  }
};
</script>

<style lang="less">
.cash {
  z-index: 1000;
  padding-bottom: 55px;
}
</style>
