<template>
    <div>
        <select name="" id="" v-model="selected" @change="getSchedule">
            <option value="0">Palun vali grupp</option>
            <option :value="group.groupId" v-for="group in groups" :key="group.groupId">{{ group.groupCode }}</option>
        </select>
    </div>
</template>
<script>
import { computed, ref } from 'vue'
import dayjs from 'dayjs/esm/index'
import weekOfYear from 'dayjs/esm/plugin/weekOfYear'
dayjs.extend(weekOfYear);
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

      return {
          groups, selected, week, getSchedule, schedule, timeTable
      }
  }

}
</script>
