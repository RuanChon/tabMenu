<template>
  <div class="nav-menu">
    <el-menu
      :default-active="currentItemId"
      class="el-menu-vertical"
      text-color="#fff"
      :default-openeds="['1']"
      active-text-color="#fff"
      v-loading="loading"
      :style="loading ? 'min-height: 100vh !important;' : ''"
    >
      <template v-for="item in menus" :key="item.id">
        <!-- 判断二级菜单 -->
        <template v-if="item.children && item.children.length">
          <el-sub-menu :index="item.id + ''">
            <template #title>
              <span style="font-weight: bold">{{ item.name }}</span>
            </template>
            <template v-for="subitem in item.children" :key="subitem.id">
              <el-menu-item :index="subitem.id + ''">
                <span>{{ subitem.name }}</span>
              </el-menu-item>
            </template>
          </el-sub-menu>
        </template>
        <template v-else>
          <el-menu-item :index="item.id + ''">
            <span>{{ item.name }}</span>
          </el-menu-item>
        </template>
      </template>
    </el-menu>
  </div>
</template>

<script setup>
import Axios from "axios"
import cryptoJS from "crypto-js/crypto-js"
import { nextTick, onMounted, ref } from "vue"

const loading = ref(true)
const currentItemId = ref("1") // 默认首页
const menus = ref([])
const interface_A = ref([])
const interface_B = ref([])

onMounted(() => {
  getData()
})

// 请求接口
const getData = async () => {
  // 接口A
  const res_A = await Axios.get(
    "https://6cxx9pggi4.execute-api.us-east-1.amazonaws.com/prod/mock/meeting-a/list",
    {
      params: {
        page_now: 1,
        page_size: 20,
      },
    }
  )
  interface_A.value = JSON.parse(decryptAES(res_A.data))

  // 接口B
  const res_B = await Axios.get(
    "https://6cxx9pggi4.execute-api.us-east-1.amazonaws.com/prod/mock/meeting-b/list",
    {
      params: {
        page_now: 1,
        page_size: 20,
      },
    }
  )
  interface_B.value = JSON.parse(decryptAES(res_B.data))

  console.log("请求A返回", interface_A.value)
  console.log("请求B返回", interface_B.value)

  nextTick(() => {
    console.log("数据获取完了")
    loading.value = false
    handleData()
  })
}

// 处理数据
const handleData = () => {
  // 合并数组
  let list = interface_A.value.data.list.concat(interface_B.value.data.list)

  // 筛选日期
  let dateArr = []
  list.forEach(item => {
    dateArr.push(new Date(item?.create_time).getDate())
  })

  dateArr = Array.from(new Set(dateArr))
  dateArr = delNaN(dateArr)
  dateArr = dateArr.sort((a, b) => {
    return b - a
  })
  console.log("dateArr", dateArr)

  dateArr.forEach(item => {
    menus.value.push({
      id: item,
      name: `${item}号集合`,
      type: item,
      parentId: 0,
      children: [],
    })
  })

  // 筛选数据
  list.forEach(item => {
    let day = new Date(item?.create_time).getDate()
    menus.value.forEach(it => {
      if (day === it.id) {
        it.children.push({
          id: item.id,
          name: item.title,
          parentId: 1,
        })
      }
    })
  })

  currentItemId.value = menus.value[0].children[0].id
}

// 解密
const decryptAES = params => {
  let keys = ""
  let decryptStr = ""
  keys = cryptoJS.enc.Utf8.parse("0000000000000000")
  decryptStr = cryptoJS.AES.decrypt(params, keys, {
    mode: cryptoJS.mode.ECB,
    padding: cryptoJS.pad.Pkcs7,
  })
  decryptStr = cryptoJS.enc.Utf8.stringify(decryptStr).toString()
  return decryptStr
}

// 删除NaN
const delNaN = arr => {
  let newArr = []
  for (var i = 0; i < arr.length; i++) {
    if (arr[i] === arr[i]) {
      newArr.push(arr[i])
    }
  }
  return newArr
}
</script>

<style scoped lang="less">
.nav-menu {
  width: 328px;
  min-height: 100vh;
  box-sizing: border-box;

  /deep/.el-menu {
    // min-height: 100vh !important;
    background-color: #1b1e28 !important;
    overflow: hidden;
  }
  .el-menu--inline {
    background-color: #1b1e28 !important;
  }
  /deep/.el-sub-menu__title {
    margin-bottom: 10px;
    border-radius: 8px;
  }
  .el-sub-menu {
    box-sizing: border-box;
    padding: 10px 20px 0 !important;

    .el-menu-item {
      background: #1b1e28;
      border: 1px solid #41454b;
      border-radius: 8px;
      padding: 0 12px !important;
      margin-bottom: 10px;
      span {
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
    }
  }
  /deep/.el-sub-menu__title:hover {
    background-color: #282b32 !important;
  }
  .el-menu-item:hover {
    background-color: #282b32 !important;
  }
  .el-menu-item.is-active {
    background-color: #41454b !important;
    position: relative;
  }
}
.el-menu-vertical {
  border: none;
}
</style>
