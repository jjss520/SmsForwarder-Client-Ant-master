<template>
  <div class="content0">
    <a-space direction="vertical">
      <a-row>
        <a-col>
          <a-button type="primary" @click="locationQuery" @keyup.enter="locationQuery">获取定位</a-button>
        </a-col>
      </a-row>
    </a-space>
    <div class="descriptions">
      <a-descriptions bordered title="设备定位">
        <a-descriptions-item label="定位时间">
          {{ data.time|defaultEmpty }}
        </a-descriptions-item>
        <a-descriptions-item label="经度">
          {{ data.longitude|defaultEmpty }}
        </a-descriptions-item>
        <a-descriptions-item label="纬度">
          {{ data.latitude|defaultEmpty }}
        </a-descriptions-item>
        <a-descriptions-item label="地址">
          {{ data.address|defaultEmpty }}
        </a-descriptions-item>
        <a-descriptions-item label="定位方式">
          {{ data.provider|defaultEmpty }}
        </a-descriptions-item>
      </a-descriptions>
    </div>
  </div>
</template>


<script>
import * as tools from "@/util/tools";
import {SECRET, SERVER_URL} from "@/store/storeKeys";


export default {
  data() {
    return {
      secret: this.$store.getters[SECRET],
      serverUrl: this.$store.getters[SERVER_URL],
      data: {}
    }
  },
  compute: {
  },
  created() {
    this.locationQuery()
  },
  methods: {
    locationQuery() {
      let timestamp = new Date().getTime();
      console.log('开始获取定位，URL:', this.serverUrl + `/location/query`);
      console.log('请求数据:', {
        "data": {},
        "timestamp": timestamp,
        "sign": tools.sign(timestamp, this.secret)
      });
      this.$axios({
        method: 'post',
        url: this.serverUrl + `/location/query`,
        data: {
          "data": {},
          "timestamp": timestamp,
          "sign": tools.sign(timestamp, this.secret)
        }
      }).then(res => {
        console.log('获取定位响应状态:', res.status, res.statusText);
        console.log('获取定位响应数据:', res.data);
        if (res.data && res.data.code === 200) {
          console.log('定位数据:', res.data.data);
          this.data = res.data.data;
          console.log('更新后的数据:', this.data);
          
          // 检查定位数据是否为空
          const isEmptyData = !this.data.time && !this.data.latitude && !this.data.longitude;
          if (isEmptyData) {
            this.$message.warning('获取到的定位数据为空，请检查设备定位设置');
          }
        } else {
          console.log('获取定位失败，响应码:', res.data ? res.data.code : '无', '消息:', res.data ? res.data.msg : '无');
          this.$message.error('获取定位失败: ' + (res.data ? res.data.msg : '未知错误'));
        }
      }).catch(err => {
        console.error('获取定位出错:', err);
        console.error('错误详细信息:', err.response ? err.response.data : err.message);
        this.$message.error('获取定位出错: ' + (err.message || '网络错误'));
      })
    }
  },
  filters: {
    defaultEmpty(val) {
      return tools.getOrDefault({val}, 'val', "")
    }
  },
  watch: {}
}
</script>
<style type="less">
.descriptions {
  margin: 20px;
}

</style>