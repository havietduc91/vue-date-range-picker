<template>
  <div class="daterangepicker-calendar">
    <div class="d-flex align-items-center">
      <div class="p-1">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToPrevMonth">
          <font-awesome-icon icon="caret-left" fixed-width />
        </button>
      </div>
      <div class="p-1 col text-center">
        {{ displayMonth.format('MM/YYYY') }}
      </div>
      <div class="p-1">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToNextMonth">
          <font-awesome-icon icon="caret-right" fixed-width />
        </button>
      </div>
    </div>
    <div class="d-flex justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in daysOfFirstWeek" :key="day.format('D')" class="col-day">{{ day.format('dd') }}</div>
    </div>

    <div class="d-flex flex-wrap justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in days" :key="day.format('M-D')" class="col-day" :class="dayClass(day)"
        @mouseover="dayMouseOver(day)" @mousedown.prevent @click="dayClick(day)">
        {{ day.format('D') }}
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { library } from '@fortawesome/fontawesome-svg-core'
import { faCaretLeft, faCaretRight } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

library.add(faCaretLeft, faCaretRight)

export default {
  props: ['calendarIndex', 'calendarCount', 'month', 'startDate', 'endDate', 'compare', 'startDateCompare', 'endDateCompare', 'step'],
  data: () => {
    return {}
  },
  computed: {
    displayMonth: function() {
      return moment.utc(this.month).add(this.calendarIndex - 1, 'month')
    },
    days: function() {
      let startDay = moment.utc(this.displayMonth).startOf('isoWeek')
      let endDay = moment.utc(this.displayMonth).endOf('month').endOf('isoWeek').startOf('day').add(1, 'day')
      let nDays = moment.duration(endDay.diff(startDay)).asDays()

      let days = []
      let day = 0
      while (day < nDays) {
        days.push(moment.utc(startDay).add(day, 'day'))
        day++
      }
      return days
    },
    daysOfFirstWeek: function() {
      return this.days.slice(0, 7)
    }
  },
  mounted() {

  },
  methods: {
    dayClass: function(day) {
      let classes = []

      // Hide days overflowing
      if (!day.isBetween(this.displayMonth, moment.utc(this.displayMonth).endOf('month'), 'days', '[]')) {
        classes.push('invisible')
      }
      // Class for days between startDate & endDate or is startDate (in case of startDate after endDate)
      if (day.isBetween(this.startDate, this.endDate, 'days', '[]') || day.isSame(this.startDate)) {
        classes.push('daterangepicker-range')
      }
      // Class for days between startDateCompare & endDateCompare or is startDateCompare (in case of startDateCompare after endDateCompare)
      if (this.compare && (day.isBetween(this.startDateCompare, this.endDateCompare, 'days', '[]') || day.isSame(this.startDateCompare))) {
        classes.push('daterangepicker-range-compare')
      }
      // Class for cursor if the step is selecting something
      if (this.step != null) {
        classes.push('daterangepicker-cursor-pointer')
      }

      if (day.isSame(this.startDate)) {
        classes.push('daterangepicker-start-date')
      }

      if (day.isSame(this.endDate)) {
        classes.push('daterangepicker-end-date')
      }

      if (day.isSame(this.startDateCompare)) {
        classes.push('daterangepicker-compare-start-date')
      }

      if (day.isSame(this.endDateCompare)) {
        classes.push('daterangepicker-compare-end-date')
      }
      return classes.join(' ')
    },
    dayMouseOver: function(day) {
      if (this.step != null) {
        this.selectDate(day)
      }
    },
    dayClick: function(day) {
      if (this.step != null) {
        this.nextStep()
      }
    },
    goToPrevMonth: function() {
      this.$emit('goToPrevMonth')
    },
    goToNextMonth: function() {
      this.$emit('goToNextMonth')
    },
    selectDate: function(date) {
      this.$emit('selectDate', date)
    },
    nextStep: function() {
      this.$emit('nextStep')
    }
  },
  watch: {},
  filters: {},
  components: { FontAwesomeIcon }
}
</script>

<style>
.daterangepicker-calendar-row {
  font-size: 14px;
  /*max-width: 230px;*/
}

.col-day {
  width: 14.28%;
  padding: 0.5rem 0;
  white-space: nowrap;
  cursor: default;
}

.daterangepicker-range {
  background-color: rgba(30,164,255,0.1) !important;
  color: #222222;
}

.daterangepicker-range-compare {
  background-color: rgba(255,124,0,0.1);
  color: #222222;
}

.daterangepicker-range.daterangepicker-range-compare {
  background: linear-gradient(0, rgba(30,164,255,0.1) 50%, rgba(255,124,0,0.1) 50%);
}

.daterangepicker-cursor-pointer {
  cursor: pointer;
}

.daterangepicker-start-date {
  background-color: #1ea4ff !important;
  color: #ffffff;
  border-top-left-radius: 4px;
  border-bottom-left-radius: 4px;
}

.daterangepicker-end-date {
  background-color: #1ea4ff !important;
  color: #ffffff;
  border-top-right-radius: 4px;
  border-bottom-right-radius: 4px;
}

.daterangepicker-compare-start-date {
  background-color: #ff8660 !important;
  color: #ffffff;
  border-top-left-radius: 4px;
  border-bottom-left-radius: 4px;
}

.daterangepicker-compare-end-date {
  background-color: #ff8660 !important;
  color: #ffffff;
  border-top-right-radius: 4px;
  border-bottom-right-radius: 4px;
}
</style>
