<template>
  <div>
    <Category :scene="scene" />
    <el-card style="margin: 10px 0px;">
      <div v-show="scene == 0">
        <el-button @click="addAttr" type="primary" size="default" icon="Plus"
          :disabled="categoryStore.c3Id ? false : true">添加属性</el-button>
        <el-table border style="margin: 10px 0px" :data="attrArr">
          <el-table-column label="序号" type="index" align="center" width="80px"></el-table-column>
          <el-table-column label="属性名称" width="120px" prop="attrName"></el-table-column>
          <el-table-column label="属性值名称">
            <template #="{ row, $index }">
              <el-tag style="margin: 5px;" v-for="(item, index) in row.attrValueList" :key="item.id">{{ item.valueName
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="120px">
            <!-- row：已有的属性对象 -->
            <template #="{ row, $index }">
              <el-button type="primary" size="small" icon="Edit" @click="updateAttr"></el-button>
              <el-button type="primary" size="small" icon="Delete"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="scene == 1">
        <!-- 添加与修改数据结构 -->
        <el-form :inline="true">
          <el-form-item label="属性名称">
            <el-input placeholder="请输入属性的名字" v-model="attrParams.attrName"></el-input>
          </el-form-item>
        </el-form>
        <el-button @click="addAttrValue" :disabled="attrParams.attrName ? false : true" type="primary" size="default"
          icon="Plus">添加属性值</el-button>
        <el-button type="primary" size="default" @click="cancel">取消</el-button>
        <el-table border style="margin:10px 0px;" :data="attrParams.attrValueList">
          <el-table-column label="序号" width="80px" type="index" align="center"></el-table-column>
          <el-table-column label="属性值名称">
            <!-- row：当前的属性对象 -->
            <template #="{ row, $index }">
              <el-input v-if="row.flag" @blur="toLook(row, $index)" placeholder="请你输入属性值名称"
                v-model="row.valueName"></el-input>
              <div v-else @click="toEdit(row)">{{ row.valueName }}</div>
            </template>
          </el-table-column>
          <el-table-column label="操作"></el-table-column>
        </el-table>
        <el-button type="primary" size="default" @click="save"
          :disabled="attrParams.attrValueList.length > 0 ? false : true">保存</el-button>
        <el-button type="primary" size="default" @click="cancel">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script setup lang='ts'>
import { watch, ref, reactive } from 'vue';
// 引入已有属性与属性值接口
import { reqAttr, reqAddOrUpdateAttr } from '@/api/product/attr'
// 获取分类的仓库
import useCategoryStore from '@/store/modules/category';
import type { AttrResponseData, Attr, AttrValue } from '@/api/product/attr/type'
import { ElMessage } from 'element-plus';
let categoryStore = useCategoryStore()
let attrArr = ref<Attr[]>([])
// 定义card组件内容切换变量
let scene = ref<number>(0) //scene=0,显示table，scene=1，展示添加与修改属性结构
//收集新增属性的数据
let attrParams = reactive<Attr>({
  attrName: "", //新增的属性的名字
  attrValueList: [ //新增的属性值数组

  ],
  categoryId: '', //三级分类的ID
  categoryLevel: 3, //代表的是三级分类
})
// 控制编辑模式与查看模式的切换
let flag = ref<boolean>(true)
// 监听仓库三级分类ID变化
watch(() => categoryStore.c3Id, () => {
  // 如果三级分类发生变化，清空上一次查询的属性和属性值
  attrArr.value = []
  // 保证有三级分类才能发请求
  if (!categoryStore.c3Id) return
  getAttr()
})

// 获取已有的属性与属性值方法
const getAttr = async () => {
  const { c1Id, c2Id, c3Id } = categoryStore
  let result: AttrResponseData = await reqAttr(c1Id, c2Id, c3Id)
  if (result.code == 200) {
    attrArr.value = result.data
  }
}

// 添加属性按钮
const addAttr = () => {
  // 每一次点击的时候先清空一下数据，在收集数据
  Object.assign(attrParams, {
    attrName: "", //新增的属性的名字
    attrValueList: [ //新增的属性值数组

    ],
    categoryId: categoryStore.c3Id, //三级分类的ID
    categoryLevel: 3, //代表的是三级分类
  })
  scene.value = 1

}

// table中的修改已有属性
const updateAttr = () => {
  scene.value = 1
}

const cancel = () => {
  scene.value = 0
}

// 添加属性值
const addAttrValue = () => {
  // 点击添加属性值按钮的时候，像数组添加一个属性值对象
  attrParams.attrValueList.push({
    valueName: "",
    flag: true
  })
}
// 保存按钮
const save = async () => {
  // 发请求
  let result = await reqAddOrUpdateAttr(attrParams)
  if (result.code == 200) {
    scene.value = 0
    ElMessage({
      type: 'success',
      message: attrParams.id ? '修改成功' : '添加成功'
    })
    // 获取全部已有的属性与属性值
    getAttr()
  } else {
    ElMessage({
      type: 'error',
      message: attrParams.id ? '修改失败' : '添加失败'
    })
  }

}

// 属性值表单元素失去焦点
const toLook = (row: AttrValue, $index: number) => {
  // 非法情况判断1，不能为空
  if (row.valueName.trim() == '') {
    // 删除掉对应属性值为空的元素
    attrParams.attrValueList.splice($index, 1)
    ElMessage({
      type: 'error',
      message: '属性值不能为空'
    })
    return
  }
  // 非法情况判断1，不能重复
  let repeat = attrParams.attrValueList.find((item) => {
    // 把当前失去焦点属性值从当前数组扣除判断
    if (item != row) {
      return item.valueName === row.valueName
    }
  })
  if (repeat) {
    ElMessage({
      type: 'error',
      message: '属性值不能重复'
    })
    return
  }
  row.flag = false
}

const toEdit = (row: AttrValue) => {
  row.flag = true
}
</script>

<style scoped></style>