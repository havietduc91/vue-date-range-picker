<template>
  <div class="d-flex daterangepicker-row">
    <!-- Right form -->
    <div class="daterangepicker-col">
      <div v-if="setRangeSelectLabel" class="set-range-select-title">{{ setRangeSelectLabel }}</div>
      <div class="form-group">
        <select class="custom-select" name="range_select" :class="compare ? 'daterangepicker-range-border' : ''" v-model="rangeSelect">
          <option v-for="(range, rangeKey) in ranges" :key="rangeKey" :value="rangeKey">{{ range.label }}</option>
          <option value="custom">Custom range</option>
        </select>
      </div>
      <div class="form-group form-inline flex-nowrap">
        <input type="text" class="form-control w-100 daterangepicker-date-input"
               ref="startDate"
               :disabled="rangeSelect !== 'custom'"
               :value="startDate | dateFormat"
               @focus="step = 'selectStartDate'" @blur="inputDate"
        >
        <span class="mx-2">
        -
        </span>
        <input type="text" class="form-control w-100 daterangepicker-date-input"
               ref="endDate"
               :disabled="rangeSelect !== 'custom'"
               :value="endDate | dateFormat"
               @focus="step = 'selectEndDate'" @blur="inputDate"
        >
      </div>
      <div class="form-group" v-if="allowCompare">
        <div class="custom-control custom-checkbox">
          <input type="checkbox" class="custom-control-input" :id="'date-range-picker-compare-' + _uid" v-model="compare">
          <label class="custom-control-label" :for="'date-range-picker-compare-' + _uid">{{ compareLabel }}</label>
        </div>
      </div>
      <div>
        <div class="form-group">
          <select class="custom-select" name="range_select_compare" :class="compare ? 'daterangepicker-range-border compare' : ''" v-model="rangeSelectCompare"
            :disabled="!compare"
            @change="onRangeSelectCompareChanged">
            <option v-for="(range, rangeKey) in compareRanges" :key="rangeKey" :value="rangeKey">{{ range.label }}</option>
            <option value="last_period">Last Period</option>
            <option value="custom">Custom range</option>
          </select>
        </div>
        <div class="form-group form-inline flex-nowrap">
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
             v-if="rangeSelectCompare === 'last_period'"
             ref="startDateCompare"
             disabled
             value=""
          >
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
             v-if="rangeSelectCompare !== 'last_period'"
             ref="startDateCompare"
             :disabled="(rangeSelectCompare !== 'custom' || !compare)"
             :value="startDateCompare | dateFormat"
             @focus="step = 'selectStartDateCompare'" @blur="inputDate"
             @keyup.enter="inputDate"
          >
          <span class="mx-2">
          -
          </span>
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
             v-if="rangeSelectCompare === 'last_period'"
             value=""
             ref="endDateCompare"
             disabled
          >
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
             ref="endDateCompare"
             v-if="rangeSelectCompare !== 'last_period'"
             :value="endDateCompare | dateFormat"
             :disabled="(rangeSelectCompare !== 'custom' || !compare)"
             @focus="step = 'selectEndDateCompare'" @blur="inputDate"
             @keyup.enter="inputDate"
          >
        </div>
      </div>
    </div>

    <!-- Calendars -->
    <div class="daterangepicker-col" v-for="calendarIndex in calendarCount" :key="calendarIndex">
      <date-range-picker-calendar
        :calendarIndex="calendarIndex"
        :calendarCount="calendarCount"
        :month="month"
        :startDate="startDate"
        :endDate="endDate"
        :compare="compare"
        :startDateCompare="startDateCompare"
        :endDateCompare="endDateCompare"
        :step="step"
        v-on:goToPrevMonth="goToPrevMonth"
        v-on:goToNextMonth="goToNextMonth"
        v-on:selectDate="selectDate"
        v-on:nextStep="nextStep"
      />
    </div>
    <div class="form-group form-inline justify-content-end mb-0 custom-btn-group">
      <button type="button" class="btn btn-light" @click="cancel">{{ cancelLabel }}</button>
      <button type="button" class="btn btn-primary ml-2" :disabled="step != null" @click="submit">{{ submitLabel }}</button>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { library } from '@fortawesome/fontawesome-svg-core'
import { faCaretRight } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import DateRangePickerCalendar from './DateRangePickerCalendar'

library.add(faCaretRight)

export default {
  props: {
    calendarCount: {
      type: Number,
      default: 2
    },
    cancelLabel: {
      type: String,
      default: 'Cancel'
    },
    submitLabel: {
      type: String,
      default: 'Cancel'
    },
    compareLabel: {
      type: String,
      default: 'Compare'
    },
    setRangeSelectLabel: {
      type: String,
      default: ''
    },
    allowCompare: {
      type: Boolean,
      default: true
    },
    ranges: {
      type: Object,
      default: function() {
        return {
          currentMonth: {
            label: 'Current month',
            startDate: moment.utc().startOf('month'),
            endDate: moment.utc().endOf('month').startOf('day')
          },
          lastMonth: {
            label: 'Last month',
            startDate: moment.utc().subtract(1, 'month').startOf('month'),
            endDate: moment.utc().subtract(1, 'month').endOf('month').startOf('day')
          }
        }
      }
    },
    compareRanges: {
      type: Object,
      default: function() {}
    },
    defaultRangeSelect: {
      type: String,
      default: 'currentMonth'
    },
    defaultRangeSelectCompare: {
      type: String,
      default: 'last_period'
    },
    defaultStartDate: {
      type: String,
      default: null
    },
    defaultEndDate: {
      type: String,
      default: null
    },
    defaultStartDateCompare: {
      type: String,
      default: null
    },
    defaultEndDateCompare: {
      type: String,
      default: null
    },
    defaultCompare: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      startDate: moment.utc(),
      endDate: moment.utc(),
      startDateCompare: '',
      endDateCompare: '',
      rangeSelect: null,
      rangeSelectCompare: 'last_period',
      compare: false,
      month: moment.utc().subtract(1, 'month').startOf('month'),
      step: null
    }
  },
  computed: {
    nextMonth: function() {
      return moment.utc(this.month).add(1, 'month')
    },
    // For multi prop watchers
    range: function() {
      return (this.startDate, this.endDate)
    },
    rangeCompare: function() {
      return (this.startDateCompare, this.endDateCompare)
    }
  },
  methods: {
    goToPrevMonth: function() {
      this.month = moment.utc(this.month).subtract(1, 'month')
    },
    goToNextMonth: function() {
      this.month = moment.utc(this.month).add(1, 'month')
    },
    selectRange: function(rangeKey) {
      let predefinedRange = false

      // Predefined ranges
      for (const _rangeKey of Object.keys(this.ranges)) {
        const range = this.ranges[_rangeKey]
        if (rangeKey === _rangeKey) {
          predefinedRange = true

          if (!this.startDate.isSame(range.startDate)) {
            this.startDate = moment.utc(range.startDate)
          }
          if (!this.endDate.isSame(range.endDate)) {
            this.endDate = moment.utc(range.endDate)
          }
        }
      }

      // Custom range
      if (!predefinedRange && this.step == null) {
        this.step = 'selectStartDate'
        const $vm = this
        setTimeout(function() {
          $vm.$refs.startDate && $vm.$refs.startDate.focus()
        }, 0)
      }
    },
    onRangeSelectCompareChanged: function(event) {
      const rangeKey = event.target.value
      this.selectRangeCompare(rangeKey)
    },
    selectRangeCompare: function(rangeKey) {
      let predefinedRange = false

      // Predefined compareRanges ranges
      if (this.compareRanges) {
        for (const _rangeKey of Object.keys(this.compareRanges)) {
          const range = this.compareRanges[_rangeKey]
          if (rangeKey === _rangeKey) {
            predefinedRange = true

            if (!this.startDateCompare || !this.startDateCompare.isSame(range.startDate)) {
              this.startDateCompare = moment.utc(range.startDate)
            }
            if (!this.endDateCompare || !this.endDateCompare.isSame(range.endDate)) {
              this.endDateCompare = moment.utc(range.endDate)
            }
          }
        }
      }

      // Custom range
      this.rangeSelectCompare = rangeKey
      if (!predefinedRange && rangeKey === 'custom') {
        this.step = 'startDateCompare'
        this.startDateCompare = moment.utc().startOf('month')
        this.endDateCompare = moment.utc().endOf('month').startOf('day')

        const $vm = this
        setTimeout(function() {
          $vm.$refs.startDateCompare && $vm.$refs.startDateCompare.focus()
        }, 0)
      } else if (rangeKey === 'last_period') {
        this.startDateCompare = ''
        this.endDateCompare = ''
      }
    },
    selectDate: function(date) {
      if (this.step === 'selectStartDate') {
        this.startDate = date
      } else if (this.step === 'selectEndDate') {
        this.endDate = date
      } else if (this.step === 'selectStartDateCompare') {
        this.startDateCompare = date
      } else if (this.step === 'selectEndDateCompare') {
        this.endDateCompare = date
      }
    },
    // Step flow for date range selections
    nextStep: function() {
      if (this.step === 'selectStartDate') {
        this.step = 'selectEndDate'
        this.$refs.endDate.focus()
      } else if (this.step === 'selectEndDate') {
        this.step = null
        this.$refs.endDate.blur()
      } else if (this.step === 'selectStartDateCompare') {
        this.step = 'selectEndDateCompare'
        this.$refs.endDateCompare.focus()
      } else if (this.step === 'selectEndDateCompare') {
        this.step = null
        this.$refs.endDateCompare.blur()
      }
    },
    // Try to update the step date from an input value
    inputDate: function(input) {
      let date = moment.utc(input.target.value, 'MM/DD/YYYY')
      if (date.isValid()) {
        this.selectDate(date)
      }
      this.nextStep()
    },
    // Submit button
    submit: function() {
      this.$emit('submit', {
        startDate: this.startDate,
        endDate: this.endDate,
        compare: this.compare,
        rangeSelect: this.rangeSelect,
        rangeSelectCompare: this.rangeSelectCompare,
        startDateCompare: this.startDateCompare,
        endDateCompare: this.endDateCompare
      })
    },
    // Cancel button
    cancel: function() {
      this.$emit('cancel')
    }
  },
  watch: {
    rangeSelect: function(rangeKey) {
      this.selectRange(rangeKey)
    }
    // rangeSelectCompare: function(rangeKey) {
    //   if (rangeKey) {
    //     this.selectRangeCompare(rangeKey)
    //   }
    // }
  },
  filters: {
    dateFormat: function(value) {
      return value ? value.format('MM/DD/YYYY') : ''
    }
  },
  created: function() {
    // Initialize ranges
    this.rangeSelect = this.defaultRangeSelect
    this.rangeSelectCompare = this.defaultRangeSelectCompare
    this.compare = this.defaultCompare
    if (this.rangeSelect === 'custom') {
      this.startDate = moment.utc().startOf('month')
      if (this.defaultEndDate) {
        this.startDate = moment.utc(this.defaultStartDate, 'YYYY/MM/DD')
      }
      this.endDate = moment.utc().endOf('month').startOf('day')
      if (this.defaultEndDate) {
        this.endDate = moment.utc(this.defaultEndDate, 'YYYY/MM/DD')
      }
    }
    if (this.rangeSelectCompare === 'custom') {
      this.startDateCompare = moment.utc().startOf('month')
      if (this.defaultStartDateCompare) {
        this.startDateCompare = moment.utc(this.defaultStartDateCompare, 'YYYY/MM/DD')
      }
      this.endDateCompare = moment.utc().endOf('month').startOf('day')
      if (this.defaultEndDateCompare) {
        this.endDateCompare = moment.utc(this.defaultEndDateCompare, 'YYYY/MM/DD')
      }
    }
  },
  components: { DateRangePickerCalendar, FontAwesomeIcon }
}
</script>

<style>
/* Custom row */
.daterangepicker-row {
  border-bottom: 1px solid #D5DAE0;
}

.daterangepicker-col {
  padding: 0.5rem;
  flex-basis: 100%;
}
.set-range-select-title {
  margin-bottom: 8px;
}

/* Make sure that the full date (YYYY-MM-DD) is displayed */
.daterangepicker-date-input {
  min-width: 120px;
}

/* Select menus border */
.daterangepicker-range-border:focus {
  box-shadow: none !important;
  border-color: #D5DAE0 !important;
}

.daterangepicker-range-border.compare:focus {
  box-shadow: none !important;
  border-color: #D5DAE0 !important;
}

/* Date input focus */
.daterangepicker-date-input:focus {
  box-shadow: none !important;
  border-color: #D5DAE0 !important;
}

.daterangepicker-date-input.compare:focus {
  border-color: #D5DAE0 !important;
  box-shadow: none !important;
}

.custom-btn-group {
  position: absolute;
  right: 0;
  bottom: 0;
}
</style>
