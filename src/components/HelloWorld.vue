<template>
    <div class="flex justify-center p-4">
        <select name="" id="" v-model="selected" @change="getSchedule" class="border border-blue-400 p-2">
            <option value="0">Palun vali grupp</option>
            <option :value="group.groupId" v-for="group in groups" :key="group.groupId">{{ group.groupCode }}</option>
        </select>
    </div>
    <div class="flex flex-col justify-center gap-4 p-4 ">
        <div v-for="(day, index) in timeTable" :key="index">
            <div class="text-center justify-between border border-indigo-400 rounded bg-gray-100">
            <span class="capitalize">{{ dayTitle(index) }}</span>
            <span>{{ date(index) }}</span>
            </div>
            <div v-for="lesson in day" :key="lesson.id" class="flex justify-between mx-4 shadow p-2 border border-indigo-200 mt-2 bg-blue-100 text-blue-900">
                <div class="text-sm">
                    <div>{{ lesson.nameEt }}</div>
                    <div class="text-sm font-bold">{{ lesson.timeStart }} - {{ lesson.timeEnd }}</div>
                </div>
                <div>
                    <div v-if="lesson.rooms != null" class="text-lg font-bold self-center">{{ lesson.rooms[0].roomCode }}</div>
                </div>
        </div>
    </div>
</template>
<script>
import { computed, ref } from 'vue'
import dayjs from 'dayjs/esm/index'
import weekOfYear from 'dayjs/esm/plugin/weekOfYear'
import et from "dayjs/esm/locale/et";
dayjs.extend(weekOfYear);
dayjs.locale(et);
export default {
  name: 'HelloWorld',
  setup(){
      const groups = ref([]);
      const selected = ref(0);
      const schedule = ref({
          timetableEvents: []
      });
      const week = dayjs().week() - dayjs('2020-09-01').week();

      (async () => {
          const res = await fetch('https://be.ta19heinsoo.itmajakas.ee/api/groups');
          groups.value = await res.json();
      })();

      const getSchedule = async () => {
          const res = await fetch(`https://be.ta19heinsoo.itmajakas.ee/api/lessons/groups=${selected.value}&weeks=${week}`);
          schedule.value = await res.json();
      }

      const timeTable = computed(() => {
         return schedule.value.timetableEvents.reduce((acc, item)=> {
              if(!(item.date in acc)){
                  acc[item.date] = []
              }
              acc[item.date].push(item);
              return acc
          }, {});
      })

      const dayTitle = (date) => {
          return dayjs(date).format('dddd');
      };

    const date = (date) => {
        return dayjs(date).format('DD.MM')
    }

      return {
          groups, selected, week, getSchedule, schedule, timeTable, dayTitle, date
      }
  }

}
</script>
