<template>
<div class="calendar__selector">
  <button class="calendar__btn material-icons" @click="selectPrevious">keyboard_arrow_left</button>
  <button class="calendar__btn" @click="selectCurrent">{{ $t('reservationToday') }}</button>
  <button class="calendar__btn material-icons" @click="selectNext">keyboard_arrow_right</button>
</div>
</template>

<script lang="ts">
import {
  Component,
  Vue,
  Prop,
  Provide,
} from 'nuxt-property-decorator';
import dayjs from "dayjs";

@Component
export default class Calendar extends Vue {
  @Prop() currentDate!: string;
  @Prop() selectedDate!: any;

  selectPrevious() {
    let newSelectedDate = dayjs(this.selectedDate).subtract(1, "month");
    this.$emit("dateSelected", newSelectedDate);
  }

  selectCurrent() {
    let newSelectedDate = dayjs(this.currentDate);
    this.$emit("dateSelected", newSelectedDate);
  }

  selectNext() {
    let newSelectedDate = dayjs(this.selectedDate).add(1, "month");
    this.$emit("dateSelected", newSelectedDate);
  }

}
</script>
