<template>
  <q-page class="row justify-left q-pa-lg">
    <div class="block-item">
      <div class="row justify-between items-center">
        <div class="text-h6">員工基本資訊</div>

        <div>
          <q-btn class="q-mr-sm" color="primary" label="新增" @click="addRow" />
          <q-btn color="white" text-color="black" label="刪除" @click="deleteBtn" />
        </div>
      </div>

      <div class="row q-mt-md q-mb-none">
        <div class="q-mr-md">
          <q-input rounded outlined clearable label="搜尋姓名" bottom-slots v-model="nameText">
            <template v-slot:append>
              <q-icon name="search" @click="search('name')" />
            </template>
          </q-input>
        </div>
        <div class="q-mr-md">
          <q-input rounded outlined clearable label="搜尋手機" bottom-slots v-model="cellphoneText">
            <template v-slot:append>
              <q-icon name="search" @click="search('cellphone')" />
            </template>
          </q-input>
        </div>
        <div class="q-mr-md">
          <q-input rounded outlined clearable label="搜尋信箱" bottom-slots v-model="emailText">
            <template v-slot:append>
              <q-icon name="search" @click="search('email')" />
            </template>
          </q-input>
        </div>
        <div class="q-mr-md">
          <q-input rounded outlined clearable label="搜尋性別" bottom-slots v-model="genderText">
            <template v-slot:append>
              <q-icon name="search" @click="search('gender')" />
            </template>
          </q-input>
        </div>
      </div>

      <QTable
        :columns="state.columns"
        :rows="state.rows"
        row-key="id"
        :selected-rows-label="getSelectedString"
        selection="multiple"
        v-model:selected="selected"
        :loading="loading"
      />
    </div>

    <!-- FIXME: 1. 最外層 div 應該不需要 -->
    <!-- FIXED -->
    <q-dialog v-model="isShow">
      <q-card style="width: 400px">
        <q-card-section class="row items-center q-pb-none">
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-h5 text-center q-pb-md">刪除</div>
          <div class="text-h6 text-center q-py-xs">是否確認刪除 {{ selectedCount }} 筆資料?</div>
        </q-card-section>

        <q-card-actions align="center" class="q-pb-xl">
          <q-btn outline class="q-mr-sm" label="取消" color="primary" v-close-popup />
          <q-btn no-caps label="確定" color="primary" v-close-popup @click="removeRow" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<!-- FIXME: 2. 改成用 Composition API + es6 語法 -->
<!-- FIXED -->
<script>
import { onMounted, reactive, ref, watch } from 'vue';
import axios from 'axios';
import QTable from 'src/components/QTable.vue';
import { format } from 'date-fns';

export default {
  name: 'IndexPage',
  components: {
    QTable,
  },
  setup() {
    //取得table資料
    const fetchData = async () => {
      try {
        const response = await axios.get('http://35.194.177.50:7777/members');
        //"birthday":"1990-2-28T15:00:00.000-07:00"：抓出T以前的字串再格式化
        state.rows = response.data.members.map((item) => ({
          id: Math.random(),
          ...item,
          birthday: format(item.birthday.split('T')[0], 'yyyy/MM/dd'),
        }));
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    //初始化加載完成
    onMounted(() => {
      fetchData();
    });

    // 欄位顯示
    const state = reactive({
      columns: [
        {
          name: 'name',
          label: '姓名',
          align: 'left',
          field: 'name',
          sortable: true,
        },
        {
          name: 'cellphone',
          label: '手機',
          align: 'left',
          field: 'cellphone',
          sortable: true,
        },
        {
          name: 'email',
          label: '信箱',
          align: 'left',
          field: 'email',
          sortable: true,
        },
        {
          name: 'gender',
          label: '性別',
          align: 'left',
          field: 'gender',
          sortable: true,
        },
        {
          name: 'birthday',
          label: '生日',
          align: 'left',
          field: 'birthday',
          sortable: true,
        },
      ],
      rows: [],
    });

    //新增
    const loading = ref(false);
    const rowCount = ref(state.rows.length);
    const addRow = () => {
      //點按時開啟loading
      loading.value = true;

      setTimeout(() => {
        //如果rows刪光，要初始化
        if (state.rows.length === 0) {
          rowCount.value = 0;
        }

        const newRow = {
          id: Math.random(),
          name: '',
          cellphone: '',
          email: '',
          gender: '',
          birthday: '',
        };
        //加在最上行
        state.rows = [newRow, ...state.rows];
        //加完後關閉loading
        loading.value = false;
      }, 500);
    };

    //勾選刪除
    const selected = ref([]);
    const selectedCount = ref(selected.value.length);
    const isShow = ref(false);
    //顯示已選幾筆資料
    const getSelectedString = () => {
      return selected.value.length === 0 ? '' : `已選 ${selected.value.length} 筆資料`;
    };
    //按下刪除鈕後顯示dialog
    const deleteBtn = () => {
      //顯示dialog
      isShow.value = true;
      //dialog上顯示選取幾筆資料
      selectedCount.value = selected.value.length;
    };
    //刪除一筆資料
    const removeRow = () => {
      loading.value = true;
      setTimeout(() => {
        selected.value.forEach((selectedRow) => {
          const index = state.rows.findIndex((row) => row.id === selectedRow.id);

          state.rows = [...state.rows.slice(0, index), ...state.rows.slice(index + 1)];
        }),
          //刪完後關閉loading
          (loading.value = false);
        //刪完清除選擇的列
        selected.value.length = 0;
      }, 500);
    };

    //搜尋
    const nameText = ref('');
    const cellphoneText = ref('');
    const emailText = ref('');
    const genderText = ref('');
    const search = (item) => {
      let filter = {};
      // FIXME: 3. 以畫面設計，filter 有可能不只一個參數，filter 要改成可接受多個搜尋條件
      switch (item) {
        case 'name':
          filter = { name: nameText.value };
          break;
        case 'cellphone':
          filter = { cellphone: cellphoneText.value };
          break;
        case 'email':
          filter = { email: emailText.value };
          break;
        case 'gender':
          filter = { gender: genderText.value };
          break;
      }
      fetchSearch(filter);
    };

    // FIXME: 4. 盡量不要在 function 裡面再包一層 function，獨立 function 比較好維護和測試，除非有特殊用途
    // FIXED
    const fetchSearch = async (filter) => {
      try {
        const response = await axios.post('http://35.194.177.50:7777/members/search', {
          filter: filter,
          sort: 'name',
        });

        state.rows = response.data.members.map((item) => ({
          id: Math.random(),
          ...item,
          birthday: format(item.birthday.split('T')[0], 'yyyy/MM/dd'),
        }));
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    //監測input輸入框是否有值，當它為空時，table回到原始狀態
    watch(
      [nameText, cellphoneText, emailText, genderText],
      ([newName, newCellphone, newEmail, newGender]) => {
        //檢查是否為空，返回true或false
        if ([newName, newCellphone, newEmail, newGender].every((val) => !val)) {
          fetchData();
        }
      }
    );

    return {
      state,
      loading,
      selected,
      selectedCount,
      isShow,
      nameText,
      cellphoneText,
      emailText,
      genderText,
      addRow,
      deleteBtn,
      removeRow,
      getSelectedString,
      search,
    };
  },
};
</script>

<style lang="scss" scoped>
.block-item {
  min-width: 400px;
}
</style>
