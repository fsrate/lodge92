<template>
<div class="calendar main__panel">
  <CalendarDateIndicator :selected-date="selectedDate" />
  <div class="calendar__body">
    <CalendarWeekdays />
    <ol class="calendar__days calendar__days--main">
      <CalendarMonthDay v-for="day in allDays" :key="day.date" :day="day" :is-today="day.date === today" :is-current-month="day.isCurrentMonth" :is-selected="day.isSelected" :is-disabled="day.isDisabled" :is-blocked="day.isBlocked"
        :is-booked="day.isBooked" :in-cart="day.inCart" @select="updateSelectedDays" />
    </ol>
  </div>
  <CalendarDateSelector :current-date="today" :selected-date="selectedDate" @dateSelected="selectDate" />
</div>
</template>

<script lang="ts">
import {
  Component,
  Vue,
  Provide,
  Watch
} from 'nuxt-property-decorator';
import Day from "@/types/Day";
import dayjs from "dayjs";
import weekday from "dayjs/plugin/weekday";
import weekOfYear from "dayjs/plugin/weekOfYear";
import CalendarDateIndicator from "@/components/CalendarDateIndicator.vue";
import CalendarDateSelector from "@/components/CalendarDateSelector.vue";
import CalendarWeekdays from "@/components/CalendarWeekdays.vue";
import CalendarMonthDay from "@/components/CalendarMonthDay.vue";

dayjs.extend(weekday);
dayjs.extend(weekOfYear);

@Component
export default class CalendarMonth extends Vue {
  @Provide() selectedDate = dayjs();
  @Provide() today = dayjs().format("YYYY-MM-DD");

  private allDays: Array < Day > = [];
  private selectedDay: Day = {
    date: '',
    isCurrentMonth: false,
    isSelected: false,
    isDisabled: false,
    isBlocked: false,
    isBooked: false,
    inCart: false,
  };

  created() {
    this.allDays = [...this.days];
    this.setBlockedDays();

    if (this.$store.getters['_days/selected'].length > 0) {
      this.setSelectedDays();
    }

    if (this.$store.getters['_days/inCart'].length > 0) {
      this.setCartDays();
    }

    (this as any).unsubscribe = this.$store.subscribe((mutation, state) => {
      if (mutation.type === '_days/resetSelected') {
        this.allDays.forEach((day: Day) => day.isSelected = false);
        this.setBlockedDays();
      }

      if (mutation.type === '_days/selected2cart') {
        this.setCartDays();
      }

      if (mutation.type === '_days/removeFromCart') {
        this.setCartDays();
      }
    });
  }

  selectDate(newSelectedDate: any) {
    this.selectedDate = newSelectedDate;
  }

  @Watch('days')
  updateDays() {
    this.allDays = this.days;

    this.allDays.filter((d1: Day) => {
      if (this.$store.getters['_days/selected'].some((d2: Day) => d1.date === d2.date) === true) {
        d1.isSelected = true;
      }

      if (this.$store.getters['_days/inCart'].some((d2: Day) => d1.date === d2.date) === true) {
        d1.inCart = true;
      }

      return this.$store.getters['_days/selected'].some((d2: Day) => d1.date === d2.date);
    });

    this.setBlockedDays();
  }

  updateSelectedDays(selectedDay: Day) {
    const allDaysIndex = this.allDays.indexOf(selectedDay);

    if (this.$store.getters['_days/selected'].filter((day: Day) => day.date === selectedDay.date).length > 0) {
      this.allDays[allDaysIndex].isSelected = false;
      this.$store.commit('_days/removeSelected', selectedDay.date);
    } else {
      const selectedDayClone = {
        ...selectedDay,
        isSelected: true
      };

      this.allDays[allDaysIndex].isSelected = true;
      this.$store.commit('_days/addSelected', selectedDayClone);
    }

    this.setBlockedDays();
  }

  setSelectedDays() {
    this.allDays.filter((d1: Day) => {
      if (this.$store.getters['_days/selected'].some((d2: Day) => d1.date === d2.date) === true) {
        d1.isSelected = true;
      }
    });
  }

  setCartDays() {
    this.allDays.filter((d1: Day) => {
      if (this.$store.getters['_days/inCart'].some((d2: Day) => d1.date === d2.date) === true) {
        d1.inCart = true;
      } else {
        d1.inCart = false;
      }
    });
  }

  setBlockedDays() {
    const sortedSelectedDays = this.sortedSelectedDays;
    const firstSelectedDay = sortedSelectedDays[0];
    const lastSelectedDay = sortedSelectedDays[sortedSelectedDays.length - 1];

    this.allDays.map((day: Day) => {
      if (!firstSelectedDay || !lastSelectedDay) {
        day.isBlocked = false;
      } else if (
        day.date === dayjs(firstSelectedDay.date).subtract(1, 'day').format("YYYY-MM-DD") ||
        day.date === dayjs(lastSelectedDay.date).add(1, 'day').format("YYYY-MM-DD") ||
        day.date === dayjs(lastSelectedDay.date).format("YYYY-MM-DD") ||
        day.date === dayjs(firstSelectedDay.date).format("YYYY-MM-DD")
      ) {
        day.isBlocked = false;
      } else {
        day.isBlocked = true;
      }
    });
  }

  getWeekday(date: any) {
    return dayjs(date).weekday();
  }

  get sortedSelectedDays() {
    return [...this.$store.getters['_days/selected']].sort((a: Day, b: Day) => (dayjs(a.date).isAfter(dayjs(b.date)) ? 1 : -1));
  }

  get days() {
    return [
      ...this.previousMonthDays,
      ...this.currentMonthDays,
      ...this.nextMonthDays
    ];
  }

  get year() {
    return this.selectedDate.format("YYYY");
  }

  get month() {
    return this.selectedDate.format("M");
  }

  get numberOfDaysInMonth() {
    return dayjs(this.selectedDate).daysInMonth();
  }

  get currentMonthDays() {
    return [...Array(this.numberOfDaysInMonth)].map((day, index) => {
      return {
        date: dayjs(`${this.year}-${this.month}-${index + 1}`).format("YYYY-MM-DD"),
        isCurrentMonth: true,
        isSelected: false,
        isDisabled: dayjs(`${this.year}-${this.month}-${index + 1}`).isBefore(dayjs().add(3, 'day').format("YYYY-MM-DD")),
        isBlocked: false,
        isBooked: false,
        inCart: false
      };
    });
  }

  get previousMonthDays() {
    const firstDayOfTheMonthWeekday = this.getWeekday(this.currentMonthDays[0].date);
    const previousMonth = dayjs(`${this.year}-${this.month}-01`).subtract(1, "month");
    const visibleNumberOfDaysFromPreviousMonth = firstDayOfTheMonthWeekday ? firstDayOfTheMonthWeekday - 1 : 6;
    const previousMonthLastMondayDayOfMonth = dayjs(this.currentMonthDays[0].date).subtract(visibleNumberOfDaysFromPreviousMonth, "day").date();

    return [...Array(visibleNumberOfDaysFromPreviousMonth)].map((day, index) => {
      return {
        date: dayjs(`${previousMonth.year()}-${previousMonth.month() + 1}-${previousMonthLastMondayDayOfMonth + index}`).format("YYYY-MM-DD"),
        isCurrentMonth: false,
        isSelected: false,
        isDisabled: dayjs(`${previousMonth.year()}-${previousMonth.month() + 1}-${previousMonthLastMondayDayOfMonth + index}`).isBefore(dayjs().format("YYYY-MM-DD")),
        isBlocked: false,
        isBooked: false,
        inCart: false
      };
    });
  }

  get nextMonthDays() {
    const lastDayOfTheMonthWeekday = this.getWeekday(`${this.year}-${this.month}-${this.currentMonthDays.length}`);
    const nextMonth = dayjs(`${this.year}-${this.month}-01`).add(1, "month");
    const visibleNumberOfDaysFromNextMonth = lastDayOfTheMonthWeekday ? 7 - lastDayOfTheMonthWeekday : lastDayOfTheMonthWeekday;

    return [...Array(visibleNumberOfDaysFromNextMonth)].map((day, index) => {
      return {
        date: dayjs(`${nextMonth.year()}-${nextMonth.month() + 1}-${index + 1}`).format("YYYY-MM-DD"),
        isCurrentMonth: false,
        isSelected: false,
        isDisabled: dayjs(`${nextMonth.year()}-${nextMonth.month() + 1}-${index + 1}`).isBefore(dayjs().format("YYYY-MM-DD")),
        isBlocked: false,
        isBooked: false,
        inCart: false
      };
    });
  }

  beforeDestroy() {
    (this as any).unsubscribe();
  }

}
</script>

<style lang="scss">
@import "../assets/scss/components/_calendar";
</style>
