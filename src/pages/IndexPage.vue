<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-input v-model="searchItem" label="查詢" />
        <!-- 根據狀態切換按鈕顯示 -->
        <q-btn color="primary" class="q-mt-md" @click="addText">
          {{ editing ? '更新' : '新增' }}
        </q-btn>
      </div>

      <q-table flat bordered ref="tableRef" :rows="filterBlockData" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ props.row[col.name] }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row, props.rowIndex)" v-for="(btn, index) in tableButtons"
                :key="index" size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">
                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>

        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted, computed } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);

const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

const editing = ref(false); // 狀態模式
const editIndex = ref(-1);    // 當前編輯索引值
const searchItem = ref('')  //新增查詢

//撈取API
const fetchData = async () => {
  try {
    const response = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = response.data;
  } catch (error) {
    console.log('錯誤', error);

  }
}
//掛載執行
onMounted(() => {
  fetchData();
})

// 新增或修改
const addText = () => {
  if (tempData.value.name !== '' && tempData.value.age !== '' && Number(tempData.value.age) && Number(tempData.value.age) > 0) {
    if (editing.value && editIndex.value !== -1) {
      //編輯
      blockData.value[editIndex.value] = { ...tempData.value };
      editing.value = false;
      editIndex.value = -1;
    } else {
      // 新增
      blockData.value.push({
        name: tempData.value.name,
        age: Number(tempData.value.age),
      });
    }
    // 清空輸入欄位
    tempData.value.name = '';
    tempData.value.age = '';
  } else {
    alert('請填寫完整資訊');
  }
};

// 處理按鈕點擊事件
async function handleClickOption(btn: btnType, data: any, index: number) {
  if (btn.status === 'edit') {
    const thisText = blockData.value[index];
    if (thisText) {
      tempData.value = { ...thisText };
      editing.value = true;  // 編輯模式
      editIndex.value = index; // 記錄當前編輯的索引
    }
  } else if (btn.status === 'delete') {
    const confirm = window.confirm('是否確定刪除該筆資料');
    if (confirm) {
      try {
        // 向後端發刪除請求
        await axios.delete('https://dahua.metcfire.com.tw/api/CRUDTest/{id}');
        // 更新數據
        blockData.value.splice(index, 1);
      } catch (error) {
        console.log('錯誤', error);
      }
    }
  }
}

//查詢
const filterBlockData = computed(() => {
  if (!searchItem.value) {
    return blockData.value;
  }
  else {
    return blockData.value.filter(item => {
      return Object.values(item).some(value => String(value).toLowerCase().includes(searchItem.value.toLowerCase()))
    })
  }
})
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
