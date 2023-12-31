<template>
  <q-table
    :title="tableName"
    v-if="!load.search"
    :rows="records"
    :columns="columns"
    row-key="id"
    :style="{ height: tableHeight }"
    separator="cell"
    rows-per-page-label="表示行数"
    no-results-label="見つからなかった..."
    no-data-label="見つからなかった..."
    :pagination="{ rowsPerPage: 0 }"
    :rows-per-page-options="[0]"
    :filter="condition"
    class="karaoke-list-table"
    ><!--sub 1/3 オプション-->
    <template v-slot:top-right>
      <div class="row q-gutter-md" style="width: 700px">
        <div>
          <q-input
            dense
            debounce="300"
            v-model="condition"
            placeholder="検索"
            style="width: 200px"
            align="left"
          >
            <template v-slot:append>
              <q-spinner
                v-model="load.search"
                v-if="load.search"
                color="primary"
                size="md"
              />
              <q-icon name="search" v-if="condition.length == 0" />
              <q-icon name="search" v-else color="primary" />
              <div class="text-caption" v-if="records.length > 0">
                {{ records.length }}
              </div>
            </template>
          </q-input>
        </div>
        <div>
          <q-select
            v-model="condition"
            :options="musicList"
            dense
            stack-label
            label="曲名"
            transition-show="jump-up"
            transition-hide="jump-up"
            style="width: 300px"
          />
        </div>
        <div>
          <q-select
            v-model="condition"
            :options="dateList"
            dense
            stack-label
            label="日付"
            transition-show="jump-up"
            transition-hide="jump-up"
            style="width: 150px"
          />
        </div>
      </div>
      <div v-if="playName != '' && playUrl != ''" class="row q-gutter-md">
        <audio controls :src="playUrl" autoplay />
        <q-field borderless class="q-pt-md" style="height: 18px">{{
          playName
        }}</q-field>
      </div>
    </template>

    <!-- sub 2/3  ヘッダー-->
    <template v-slot:header="props">
      <q-tr :props="props">
        <q-th>再生</q-th>
        <q-th v-for="col in props.cols" :key="col.name" :props="props">
          {{ col.label }}
        </q-th>
      </q-tr>
    </template>
    <!-- sub 3/3  アイテム-->
    <template v-slot:body="props">
      <q-tr :props="props">
        <q-td>
          <a
            :href="api.apiEndpoint() + '/karaoke/download?id=' + props.row.id"
            @click.prevent="
              console.log('play', props.row);
              playUrl =
                api.apiEndpoint() + '/karaoke/download?id=' + props.row.id;
              playName = props.row.fileName;
            "
            class="play-btn"
            ><q-icon name="play_circle" color="primary" size="sm"></q-icon
          ></a>
        </q-td>
        <q-td
          v-for="col in props.cols"
          :key="col.name"
          :props="props"
          style="white-space: normal; text-align: left"
        >
          {{ col.value }}
        </q-td>
      </q-tr>
    </template></q-table
  >
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { useKaraokeListModel } from 'src/models/KaraokeListModels';
import api from 'src/api/KaraokeListApi';
export default defineComponent({
  name: 'table-karaoke-list',
  props: {
    modelValue: {
      type: String,
      required: true,
    },
    label: {
      type: String,
      required: false,
      default: 'カラオケ音楽堂',
    },
    height: {
      type: Number,
      required: false,
      default: 700,
    },
  },
  setup(props) {
    const { columns, load, records, search, selectId, dateList, musicList } =
      useKaraokeListModel();

    search();
    return {
      condition: ref(props.modelValue),
      tableName: ref(props.label),
      tableHeight: ref(props.height + 'px'),
      columns,
      load,
      records,
      search,
      api,
      selectId,
      playUrl: ref(''),
      playName: ref(''),
      dateList,
      musicList,
    };
  },
});
</script>
<style>
.playground {
  padding: 0.5em 1em;
  margin: 2em 0;
  font-weight: bold;
  background: #fff;
  border: solid 3px #5c5e5f; /*線*/
  border-radius: 10px; /*角の丸み*/
}
.playground p {
  margin: 0;
  padding: 0;
}
/*テーブルのstyle */
.karaoke-list-table {
  max-width: 700px;
}

.karaoke-list-table .q-table__top,
.karaoke-list-table .q-table__bottom,
.karaoke-list-table thead tr:first-child th {
  /* bg color is important for th; just specify one */
  background-color: white;
}

.karaoke-list-table thead tr th {
  position: sticky;
  z-index: 1;
}

.karaoke-list-table thead tr:first-child th {
  top: 0;
}

/* this is when the loading indicator appears */
.karaoke-list-table.q-table--loading thead tr:last-child th {
  /* height of all previous header rows */
  top: 48px;
}

/* prevent scrolling behind sticky top row on focus */
.karaoke-list-table tbody {
  /* height of all previous header rows */
  scroll-margin-top: 48px;
}
</style>
