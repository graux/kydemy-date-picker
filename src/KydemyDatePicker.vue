<template>
    <div class="kydemy-datetime-picker"
         :class="[`is-${sizeClass}`, {'is-date-picker': dateEnabled, 'is-time-picker': timeEnabled}]">
        <portal-target :name="'portal-inline-'+componentUID"></portal-target>
        <transition name="fade" :duration="300" v-if="modal && showDateTimePicker">
            <div class="modal is-active kydemy-date-picker-modal">
                <div class="modal-background"></div>
                <div class="modal-content">
                    <portal-target :name="'portal-modal-'+componentUID"></portal-target>
                    <button class="button is-primary is-rounded modal-close-button" @click="showDateTimePicker=false">
                        <span class="icon">
                            <i class="fa fa-times"></i>
                        </span>
                    </button>
                </div>
            </div>
        </transition>
        <div class="dropdown is-right" :class="{'is-active': showDateTimePicker, 'is-up': isUp}" v-if="inline !== true">
            <div class="dropdown-trigger">
                <p class="control" :class="{'has-icons-left': iconLeft, 'has-icons-right': iconRight}">
                    <input type="text" readonly class="input" :class="[`is-${sizeClass}`]" v-model="displayDateFormat"
                           :placeholder="placeholder"
                           @click="showDateTimePicker = !showDateTimePicker"/>
                    <span v-if="iconLeft" class="icon is-left" :class="[`is-${sizeClass}`]">
                        <i :class="iconLeft"></i>
                    </span>
                    <span v-if="iconRight" class="icon is-right" :class="[`is-${sizeClass}`]">
                        <i :class="iconRight"></i>
                    </span>
                </p>
            </div>
            <div class="dropdown-menu" role="menu">
                <div class="dropdown-content">
                    <portal-target :name="'portal-dropdown-'+componentUID"></portal-target>
                </div>
            </div>
        </div>
        <portal :to="targetPortal">
            <div class="kydemy-date-picker" v-if="dateEnabled !== false" :class="[`is-${displayMode}`]">
                <div class="card">
                    <div class="card-header calendar-header">
                        <div class="card-header-title">
                            <nav class="level level-year is-mobile">
                                <div class="level-left">
                                    <a @click="prevYear">
                                <span class="icon has-text-white">
                                  <i class="fa fa-angle-left"></i>
                                </span>
                                    </a>
                                </div>
                                <div class="level-item">
                                    <button class="button is-primary" @click="selYear">
                                        <span class="has-text-white">{{ displayDate.format('YYYY') }}</span>
                                    </button>
                                </div>
                                <div class="level-right">
                                    <a @click="nextYear">
                                    <span class="icon has-text-white">
                                      <i class="fa fa-angle-right"></i>
                                    </span>
                                    </a>
                                </div>
                            </nav>
                            <nav class="level level-month is-mobile">
                                <div class="level-left">
                                    <div class="level-item">
                                        <a @click="prevMonth">
                                        <span class="icon is-large has-text-white">
                                          <i class="fa fa-lg fa-chevron-circle-left"></i>
                                        </span>
                                        </a>
                                    </div>
                                </div>
                                <div class="level-item">
                                    <button class="button is-large is-primary" @click="selMonth">
                                        <span class="title is-uppercase has-text-white">{{ displayDate.format('MMMM') }}</span>
                                    </button>
                                </div>
                                <div class="level-right">
                                    <div class="level-item">
                                        <a @click="nextMonth">
                                        <span class="icon is-large has-text-white">
                                            <i class="fa fa-lg fa-chevron-circle-right"></i>
                                        </span>
                                        </a>
                                    </div>
                                </div>
                            </nav>
                        </div>
                    </div>
                    <div class="card-content">
                        <div class="content-wrapper is-relative">
                            <transition-group name="fade" :duration="{ enter: 200, leave: 100 }" mode="out-in">
                                <div v-if="displayMode === 'day'" :key="'displayMode_day'">
                                    <div class="columns is-variable is-1 is-mobile week-headers is-marginless is-paddingless">
                                        <div class="column week-day is-capitalized" v-for="day in weekDays">
                                            {{day.short}}
                                        </div>
                                    </div>
                                    <div class="columns is-variable is-1 is-marginless is-paddingless is-mobile kydemy-calendar is-multiline">
                                        <div v-for="day in renderDays" :key="day.iso"
                                             class="column week-day"
                                             :class="[day.classes]">
                                            <button class="is-white button is-rounded is-paddingless"
                                                    :class="{'is-primary': day.selected, 'is-disabled': day.disabled}"
                                                    :disabled="day.disabled"
                                                    @click="selectDay(day)">
                                                {{ day.mnt.format('D')}}
                                            </button>
                                        </div>
                                    </div>
                                </div>
                                <div class="kydemy-year-picker" v-if="displayMode==='year'"
                                     :key="'displayMode_year'">
                                    <div class="columns is-variable is-1 is-multiline is-mobile">
                                        <div class="column is-one-fifth is-year" v-for="year in displayYears"
                                             :key="'year_'+year">
                                            <button class="button is-white is-normal is-size-6"
                                                    @click="selectYear(year)"
                                                    :class="{'is-primary': displayDate.year() === year }">
                                            <span class="icon" v-if="year === -1">
                                                <i class="fa fa-arrow-circle-left"></i>
                                            </span>
                                                <span class="icon" v-else-if="year === 1">
                                                <i class="fa fa-arrow-circle-right"></i>
                                            </span>
                                                <span v-else>{{ year }}</span>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                                <div class="kydemy-month-picker" v-if="displayMode==='month'"
                                     :key="'displayMode_month'">
                                    <div class="columns is-multiline is-mobile">
                                        <div class="column is-one-third is-month"
                                             v-for="(month,index) in displayMonths"
                                             :key="'month_'+index">
                                            <button class="button is-white is-medium is-uppercase is-size-6"
                                                    @click="selectMonth(month)"
                                                    :class="{'is-primary': month.mnt.isSame(displayDate,'month')}">
                                                {{ month.name }}
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </transition-group>
                        </div>
                    </div>
                </div>
            </div>
            <div class="kydemy-time-picker" v-if="timeEnabled !== false && displayMode === 'day'">
                <div class="card">
                    <div class="card-content">
                        <transition-group name="fade" :duration="{ enter: 200, leave: 100 }" mode="out-in">
                            <div class="kydemy-time-picker" :key="'time_picker'"
                                 v-if="timeDisplayMode === 'time'">
                                <div class="columns is-mobile is-paddingless is-marginless is-variable is-0">
                                    <div class="column column is-paddingless">
                                        <button class="button is-white" @click="timeDisplayMode = 'hours'">
                                            <span v-if="is24h">{{ displayDate.format('HH') }}</span>
                                            <span v-else>{{ displayDate.format('hh') }}</span>
                                        </button>
                                    </div>
                                    <div class="column is-narrow column is-paddingless"><span
                                            class="separator">:</span>
                                    </div>
                                    <div class="column is-paddingless">
                                        <button class="button is-white" @click="timeDisplayMode = 'minutes'">
                                            {{ displayDate.format('mm') }}
                                        </button>
                                    </div>
                                    <div class="column is-narrow column is-paddingless" v-if="!is24h">
                                        <div class="field has-addons">
                                            <p class="control">
                                                <button class="button is-white" @click="setTimeAM"
                                                        :class="{'is-primary': displayDate.format('A') === 'AM'}">
                                                    <span class="is-size-7">AM</span>
                                                </button>
                                            </p>
                                            <p class="control">
                                                <button class="button is-white" @click="setTimePM"
                                                        :class="{'is-primary': displayDate.format('A') === 'PM'}">
                                                    <span class="is-size-7">PM</span>
                                                </button>
                                            </p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="kydemy-hour-picker" :class="{'is-24-hour': is24h, 'is-12-hour': !is24h}"
                                 :key="'hour_picker'" v-if="timeDisplayMode === 'hours'">
                                <div class="columns is-marginless is-paddingless is-variable is-1 is-multiline is-mobile">
                                    <div class="column" v-for="hour in displayHours">
                                        <button class="button is-white" @click="selectHour(hour)"
                                                :class="{'is-primary': displayDate.format((is24h ? 'HH' : 'hh')) === hour }">
                                            {{ hour }}
                                        </button>
                                    </div>
                                </div>
                            </div>
                            <div class="kydemy-minute-picker" :key="'minute_picker'"
                                 v-if="timeDisplayMode === 'minutes'">
                                <div class="columns is-marginless is-paddingless is-variable is-1 is-multiline is-mobile">
                                    <div class="column is-minute" v-for="minute in displayMinutes">
                                        <button class="button is-white" @click="selectMinute(minute)"
                                                :class="{'is-primary': displayDate.format('mm') === minute }">
                                            {{ minute }}
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </transition-group>
                    </div>
                </div>
            </div>
        </portal>
    </div>
</template>
<script>
import moment from 'moment'
import business from 'moment-business'

export default {
  props: {
    value: Object | Array | String,
    locale: {type: String, required: true},
    startDate: Object | String,
    timeEnabled: {type: Boolean, default: false},
    dateEnabled: {type: Boolean, default: true},
    is24h: {type: Boolean, default: false},
    minDate: Object | String,
    maxDate: Object | String,
    enabledDates: Object,
    dimWeekends: {type: Boolean, default: false},
    highlighted: Array,
    disabled: Array,
    disabledControl: Boolean,
    range: {type: Boolean, default: false},
    multiple: {type: Boolean, default: false},
    formatFunction: Function,
    formatPattern: String,
    size: String,
    autoClose: {type: Boolean, default: false},
    interactive: {type: Boolean, default: false},
    inline: {type: Boolean, default: false},
    modal: {type: Boolean, default: false},
    placeholder: String,
    iconLeft: String,
    iconRight: String,
    isUp: {type: Boolean, default: false}
  },
  data () {
    return {
      showDateTimePicker: false,
      displayDate: null,
      weekDays: null,
      renderDays: [],
      sizeClass: null,
      displayMode: 'day',
      timeDisplayMode: 'time',
      displayMonths: null,
      displayYear: null,
      displayYears: null,
      displayHours: null,
      displayMinutes: null,
      selectedDays: [],
      selectedTime: null,
      targetPortal: 'portal-dropdown',
      oldDisplayDate: null,
      oldSelectedDays: [],
      lastEmitedValue: null
    }
  },
  methods: {
    selectDay: function (selDay) {
      const selDate = selDay.iso
      if (this.timeEnabled === true && (!self.selectedTime || self.selectedTime.length > 0)) {
        this.selectedTime = this.displayDate.format('HH:mm')
      }
      this.$emit('day-selected', selDay.mnt.clone())
      if (this.range === true) {
        if (this.selectedDays.length === 1) {
          let currDate = moment(this.selectedDays[0])
          if (currDate.isBefore(selDay.mnt)) {
            this.selectedDays.push(selDate)
          } else {
            this.selectedDays = [selDate, this.selectedDays[0]]
          }
        } else {
          this.selectedDays = [selDate]
        }
        if (this.selectedDays.length === 2) {
          this.emitSelectedDays()
          if (this.autoClose === true) {
            this.showDateTimePicker = false
          }
        }
        this.redraw()
      } else {
        this.displayDate = selDay.mnt.clone()
        if (this.multiple === true) {
          const selDateIndex = this.selectedDays.indexOf(selDate)
          if (selDateIndex >= 0) {
            this.selectedDays.splice(selDateIndex, 1)
          } else {
            this.selectedDays.push(selDate)
          }
          this.emitSelectedDays()
        } else {
          this.selectedDays = [selDate]
          this.emitSelectedDays()
          if (this.autoClose === true) {
            this.showDateTimePicker = false
          }
        }
      }
    },
    emitSelectedDays () {
      let value = this.getSelectedDays()
      const valueStr = JSON.stringify(value)
      if (this.lastEmitedValue || this.lastEmitedValue !== valueStr) {
        this.$emit('input', value)
        this.lastEmitedValue = valueStr
      }
    },
    nextMonth () {
      this.displayDate = this.displayDate.clone().add(1, 'month')
    },
    prevMonth () {
      this.displayDate = this.displayDate.clone().subtract(1, 'month')
    },
    nextYear () {
      this.displayDate = this.displayDate.clone().add(1, 'year')
    },
    prevYear () {
      this.displayDate = this.displayDate.clone().subtract(1, 'year')
    },
    redraw (newDate) {
      if (newDate === undefined) {
        newDate = this.displayDate
      }
      let isoDate = newDate.toISOString(true)
      const selDays = this.selectedDays.join(',')
      if (isoDate !== this.oldDisplayDate || this.oldSelectedDays !== selDays) {
        this.oldDisplayDate = isoDate
        this.oldSelectedDays = selDays
        const calendarStart = newDate.clone().startOf('month').weekday(0)
        const calendarEnd = newDate.clone().endOf('month').weekday(6)
        const today = moment()

        let day = calendarStart.clone()
        let days = []
        let classes = ''
        let selected = null
        while (day.isSameOrBefore(calendarEnd, 'day')) {
          isoDate = day.format('YYYY-MM-DD')
          classes = ''
          selected = this.isDaySelected(isoDate)
          if (!business.isWeekDay(day)) {
            classes += ' is-weekend'
          }
          if (!day.isSame(newDate, 'month')) {
            classes += ' is-adjacent'
          }
          if (day.isSame(today, 'day')) {
            classes += ' is-today'
          }
          if (selected) {
            classes += ' is-selected'
          }
          if (this.isDayHighlighted(day)) {
            classes += ' is-hightlighted'
          }
          if (this.range) {
            if (this.isDayInRange(day)) {
              classes += ' in-range'
            } else if (this.selectedDays.length === 2) {
              if (day.isSame(this.selectedDays[0])) {
                classes += ' range-start'
              } else if (day.isSame(this.selectedDays[1])) {
                classes += ' range-end'
              }
            }
          }

          days.push({
            mnt: day,
            selected: selected,
            classes: classes.trim(),
            disabled: this.isDayDisabled(day),
            iso: isoDate
          })
          day = day.clone().add(1, 'day')
        }
        this.renderDays = days
      }
    },
    isDaySelected: function (iso) {
      if (this.selectedDays.length > 0) {
        for (let index = 0; index < this.selectedDays.length; index++) {
          if (iso === this.selectedDays[index]) {
            return true
          }
        }
      }
      return false
    },
    isDayInRange: function (day) {
      if (this.selectedDays.length === 2) {
        let d1 = moment(this.selectedDays[0])
        let d2 = moment(this.selectedDays[1])
        return day.isAfter(d1) && day.isBefore(d2)
      }
      return false
    },
    isDayDisabled: function (day) {
      if (this.minDate) {
        if (day.isBefore(this.minDateComputed)) {
          return true
        }
      }
      if (this.maxDate) {
        if (day.isAfter(this.maxDateComputed)) {
          return true
        }
      }
      if (this.disabled && this.disabled.length > 0) {
        for (let index = 0; index < this.disabled.length; index++) {
          if (day.isSame(this.disabled[index], 'day')) {
            return true
          }
        }
      }
      return false
    },
    isDayHighlighted: function (day) {
      if (this.highlighted && this.highlighted.length > 0) {
        for (let index = 0; index < this.highlighted.length; index++) {
          if (day.isSame(this.highlighted[index], 'day')) {
            return true
          }
        }
      }
      return false
    },
    getMomentFromField: function (field) {
      if (field) {
        if (typeof field === 'string') {
          switch (field) {
            case 'today':
              return moment()
            case 'yesterday':
              return moment().subtract(1, 'day')
            case 'tomorrow':
              return moment().add(1, 'day')
            default:
              return moment(field)
          }
        } else if (typeof field === 'object') {
          return field
        }
      }
      return null
    },
    selMonth: function () {
      if (this.displayMode === 'month') {
        this.displayMode = 'day'
      } else {
        this.displayMode = 'month'
      }
    },
    selYear: function () {
      if (this.displayMode === 'year') {
        this.displayMode = 'day'
      } else {
        this.displayMode = 'year'
      }
    },
    selectMonth: function (month) {
      this.displayDate = this.displayDate.clone().month(month.index)
      this.displayMode = 'day'
    },
    selectYear: function (year) {
      if (year === -1) {
        this.displayYear -= 22
      } else if (year === 1) {
        this.displayYear += 22
      } else {
        this.displayDate = this.displayDate.clone().year(year)
        if (this.interactive === true) {
          this.displayMode = 'month'
        } else {
          this.displayMode = 'day'
        }
      }
    },
    setTimeAM: function () {
      if (!this.isAM) {
        this.displayDate = this.displayDate.clone().subtract(12, 'hours')
        this.emitSelectedTime()
      }
    },
    setTimePM: function () {
      if (this.isAM) {
        this.displayDate = this.displayDate.clone().add(12, 'hours')
        this.emitSelectedTime()
      }
    },
    emitSelectedTime: function () {
      this.$emit('time-selected', this.displayDate.clone())
      this.selectedTime = this.displayDate.format('HH:mm')
      if (this.selectedDays.length === 0) {
        this.$emit('input', [this.displayDate.clone()])
      } else {
        this.emitSelectedDays()
      }
    },
    selectHour: function (hour) {
      let hourNumber = parseInt(hour)
      if (this.is24h === false && !this.isAM) {
        hourNumber += 12
      }
      this.displayDate = this.displayDate.clone().hours(hourNumber)
      this.timeDisplayMode = 'time'
      this.emitSelectedTime()
    },
    selectMinute: function (minute) {
      this.displayDate = this.displayDate.clone().minutes(parseInt(minute))
      this.timeDisplayMode = 'time'
      this.emitSelectedTime()
    },
    getSelectedDays: function () {
      if (this.selectedDays.length > 0) {
        const self = this
        let selDays = this.selectedDays.map((iso) => {
          let dateTime = iso
          if (this.timeEnabled) {
            dateTime += ' ' + self.selectedTime
          }
          return moment(dateTime)
        })

        if (typeof this.value === 'string') {
          selDays = selDays.map((sd) => { return sd.toISOString(true) })
        }

        if (this.multiple === false && this.range === false) {
          return selDays[0]
        } else {
          return selDays
        }
      }
      return null
    },
    onKeyDown: function (event) {
      if (event.keyCode === 27) {
        this.showDateTimePicker = false
      }
    }
  },
  computed: {
    componentUID: function () {
      return this._uid
    },
    minDateComputed: function () {
      return this.getMomentFromField(this.minDate)
    },
    maxDateComputed: function () {
      return this.getMomentFromField(this.maxDate)
    },
    isAM: function () {
      return this.displayDate.format('A').toUpperCase() === 'AM'
    },
    displayDateFormat: function () {
      const self = this
      let dates = []
      let formatted = []
      if (this.selectedDays.length > 0) {
        dates = this.selectedDays.map((d) => {
          if (this.timeEnabled && self.selectedTime && self.selectedTime.length > 0) {
            return moment(d + ' ' + self.selectedTime)
          } else {
            return moment(d)
          }
        })
      } else if (this.selectedTime !== null && this.timeEnabled && this.dateEnabled === false) {
        dates = [this.displayDate]
      } else {
        dates.push(this.displayDate)
      }

      if (this.displayDate !== null) {
        if (this.formatPattern !== undefined && this.formatPattern !== null && this.formatPattern.length > 0) {
          formatted = dates.map((d) => { return d.format(this.formatPattern) })
        } else {
          let inputFormat = ''
          if (this.dateEnabled) {
            if (this.timeEnabled) {
              inputFormat += 'D MMM YYYY'
            } else {
              inputFormat += 'dddd D MMMM, YYYY'
            }
          }
          if (this.timeEnabled) {
            if (this.is24h) {
              inputFormat += ' HH:mm'
            } else {
              inputFormat += ' h:mm A'
            }
          }
          inputFormat = inputFormat.trim()
          formatted = dates.map((d) => { return d.format(inputFormat) })
        }
      }
      return formatted.join(', ')
    }
  },
  created: function () {
    moment.locale(this.locale)
    const self = this
    if (this.inline) {
      this.targetPortal = 'portal-inline-' + this.componentUID
    } else if (this.modal) {
      this.targetPortal = 'portal-modal-' + this.componentUID
    } else {
      this.targetPortal = 'portal-dropdown-' + this.componentUID
    }
    let newDisplayDate = null
    if (this.value) {
      if (typeof this.value === 'string') {
        newDisplayDate = moment(this.value)
      } else {
        newDisplayDate = this.value.clone()
      }
      this.selectedTime = newDisplayDate.format('HH:mm')
    } else if (this.startDate !== undefined && this.startDate !== null) {
      if (typeof this.startDate === 'string') {
        newDisplayDate = moment(this.startDate)
      } else {
        newDisplayDate = this.startDate.clone()
      }
    }
    if (newDisplayDate === null) {
      newDisplayDate = moment()
    }

    if (this.timeEnabled === true && this.selectedTime !== null && this.selectedTime.length > 0) {
      const tokens = this.selectedTime.split(':')
      newDisplayDate = newDisplayDate.clone().hours(parseInt(tokens[0])).minutes(parseInt(tokens[1]))
    }
    this.displayDate = newDisplayDate
    if (this.value) {
      this.selectedDays.push(newDisplayDate.format('YYYY-MM-DD'))
    }

    if (this.interactive === true) {
      this.displayMode = 'year'
    }

    if (['small', 'normal', 'medium', 'large'].indexOf(this.size) >= 0) {
      this.sizeClass = this.size
    } else {
      this.sizeClass = 'normal'
    }

    let currentWeekDay = moment().weekday(0)
    let day = null
    this.weekDays = Array.apply(0, Array(7)).map(function (_, i) {
      day = {index: i, name: currentWeekDay.format('dddd'), short: currentWeekDay.format('ddd')}
      currentWeekDay.add('1', 'day')
      return day
    })
    let currentMonth = moment({month: 0})
    let month = null
    this.displayMonths = Array.apply(0, Array(12)).map(function (_, i) {
      month = {index: i, name: currentMonth.format('MMMM'), short: currentMonth.format('MMM'), mnt: currentMonth}
      currentMonth = currentMonth.clone().add('1', 'month')
      return month
    })
    const hourFormat = (self.is24h ? 'HH' : 'hh')
    this.displayHours = Array.apply(0, Array((this.is24h ? 24 : 12))).map(function (_, i) {
      return moment({hour: i + 1}).format(hourFormat)
    })
    this.displayMinutes = Array.apply(0, Array(12)).map(function (_, i) {
      return moment({minute: i * 5}).format('mm')
    })
    if (this.$eventHub) {
      this.$eventHub.$on('expand-dropdown', function (compUID) {
        if (self.showDateTimePicker && self._uid !== compUID) {
          self.showDateTimePicker = false
        }
      })
    }
  },
  watch: {
    displayDate: function (newDate) {
      this.redraw(newDate)
      this.displayYear = parseInt(newDate.format('YYYY'))
    },
    displayYear: function (newYear) {
      let years = [-1]
      const maxYear = newYear + 11
      for (let index = newYear - 11; index <= maxYear; index++) {
        years.push(index)
      }
      years.push(1)
      this.displayYears = years
    },
    selectedTime: function (val) {
      if (this.timeEnabled === true) {
        const tokens = val.split(':')
        this.displayDate = this.displayDate.clone().hours(parseInt(tokens[0])).minutes(parseInt(tokens[1]))
      }
    },
    selectedDays: function (val) {
      if (val.length > 0) {
        if (this.timeEnabled === true) {
          if (this.selectedTime && this.selectedTime.length > 0) {
            this.displayDate = moment(val[0] + ' ' + this.selectedTime)
          } else {
            this.displayDate = moment(val[0]).hours(this.displayDate.hours()).minutes(this.displayDate.minutes())
          }
        } else {
          this.displayDate = moment(val[0])
        }
      }
    },
    value: function (newDate) {
      let newSelDay = null
      if (newDate !== null) {
        if (typeof newDate === 'string') {
          newSelDay = moment(newDate)
        } else {
          newSelDay = newDate.clone()
        }
        const iso = newSelDay.format('YYYY-MM-DD')
        if (this.multiple === true) {
          this.selectedDays.push(iso)
        } else {
          this.selectedDays = [iso]
        }
        this.selectedTime = newSelDay.format('HH:mm')
      } else {
        this.selectedDays = []
      }
      this.redraw()
    },
    disabled: function () {
      this.redraw()
    },
    showDateTimePicker: function (newVal) {
      if (newVal === true) {
        window.addEventListener('keydown', this.onKeyDown)
        if (this.$eventHub) {
          this.$eventHub.$emit('expand-dropdown', this._uid)
        }
      } else {
        window.removeEventListener('keydown', this.onKeyDown)
      }
    }
  }
}
</script>
<style lang="scss">
    @import '~bulma/sass/utilities/_all';

    .kydemy-datetime-picker {
        @media (max-width: 768px) {
            margin: 0 auto;
        }
        .dropdown {
            .dropdown-menu {
                transition: top 0.3s, bottom 0.3s, visibility 0s, opacity 0.3s;
                top: 140%;
                opacity: 0.2;
                display: block;
                visibility: hidden;
            }
            &.is-active .dropdown-menu {
                top: 100%;
                opacity: 1;
                display: block;
                visibility: visible;
            }
            &.is-up {
                .dropdown-menu {
                    top: auto;
                    bottom: 200%;
                }
                &.is-active .dropdown-menu {
                    bottom: 100%;
                    top: auto;
                    margin-top: 0;
                    margin-bottom: 1rem;
                }
            }
        }

        .kydemy-date-picker-modal {
            .modal-content {
                overflow: visible;
                .modal-close-button {
                    position: absolute;
                    top: -2rem;
                    right: -3rem;
                    @media (max-width: 639px) {
                        right: 0;
                        top: -3rem;
                    }
                }
            }
        }

        &.is-time-picker {
            .dropdown .dropdown-menu:after {
                border-bottom-color: $white;
            }
        }
        &.is-date-picker {
            .dropdown .dropdown-menu:after {
                border-bottom-color: $primary;
            }
        }
        .dropdown {
            .dropdown-menu {
                left: auto;
                right: 0;
                padding: 0;
                margin-top: 0.75rem;
                .dropdown-content {
                    padding: 0;
                }
            }
            .dropdown-menu:after, .dropdown-menu:before {
                bottom: 100%;
                left: 90%;
                border: solid transparent;
                content: " ";
                height: 0;
                width: 0;
                position: absolute;
                pointer-events: none;
            }
            &.is-up {
                .dropdown-menu:after, .dropdown-menu:before {
                    bottom: auto;
                    top: 100%;
                }
            }
            .dropdown-menu:after {
                border-color: rgba(#999999, 0);
                border-width: 10px;
                margin-left: -10px;
            }
            .dropdown-menu:before {
                border-color: rgba(#999999, 0);
                border-bottom-color: rgba(#999999, 0.3);
                border-width: 12px;
                margin-left: -12px;
            }
            &.is-up {
                .dropdown-menu:after {
                    border-bottom-color: transparent;
                    border-top-color: $white;
                }
                .dropdown-menu:before {
                    border-bottom-color: transparent;
                    border-top-color: rgba(#999999, 0.3);
                }
            }
        }
        &.is-normal {
            max-width: 23rem;
            min-width: 9rem;
            .dropdown .dropdown-content {
                min-width: 22rem;
            }
        }
        &.is-small {
            max-width: 20rem;
            min-width: 8rem;
            .dropdown .dropdown-content {
                min-width: 10rem;
            }
        }
        &.is-large {
            max-width: 40rem;
            min-width: 16rem;
            .dropdown .dropdown-content {
                min-width: 20rem;
            }
        }

        @media (max-width: 639px) {
            .dropdown {
                .dropdown-menu {
                    left: -10%;
                    &:before, &:after {
                        left: 55%;
                    }
                    .dropdown-content {
                        max-width: 110%;
                        min-width: 110%;
                        width: 110%;
                        .card-content {
                            padding: 0.5rem 0.2rem;
                        }
                    }
                }
            }
        }

        .kydemy-date-picker {
            .card-content {
                overflow: hidden;
                background: $white;
                padding-top: 1rem;
                padding-bottom: 1rem;
                .columns {
                    margin-top: 0;
                    margin-bottom: 0;
                }
            }
            .card {
                max-height: 0;
                transition: max-height 0.15s;
                overflow: hidden;
            }
            &.is-month .card {
                max-height: 24rem;
            }
            &.is-year .card {
                max-height: 23rem;
            }
            &.is-day .card {
                max-height: 27rem;
            }
        }

        .kydemy-date-picker {
            .week-headers {
                .week-day {
                    color: rgba($cyan, 0.54);
                }
            }
            .week-day {
                text-align: center;
                flex: none;
                width: 14.285714286%;
                padding-top: 0rem;
                padding-bottom: 0rem;
                margin-top: 0rem;
                margin-bottom: 0rem;
                .button {
                    width: 100%;
                    text-align: center;
                    font-weight: bold;
                    padding: 0.25rem;
                    &:active, &:focus {
                        outline: none;
                    }
                    &::-moz-focus-inner {
                        border: 0;
                    }
                }
                &.is-weekend {
                    .button {
                        font-weight: normal;
                        color: $grey-dark;
                    }
                }
                &.is-adjacent {
                    .button {
                        font-weight: normal;
                        color: $grey-lighter;
                    }
                }
                &.is-today {
                    .button {
                        border: solid 1px $primary;
                    }
                }
                &.is-hightlighted {
                    .button {
                        background-color: rgba($primary, 0.1);
                    }
                }
                &.in-range {
                    background-color: rgba($primary, 0.2);
                    .button {
                        background-color: transparent;
                        color: rgba($primary, 0.6);
                        font-weight: normal;
                    }
                }
                &.is-selected {
                    position: relative;
                    .button {
                        color: $white;
                    }
                    &.range-start::before, &.range-end::before {
                        background-color: rgba($primary, 0.2);
                        width: 50%;
                        height: 100%;
                        display: block;
                        position: absolute;
                        content: "";
                        top: 0;
                    }
                    &.range-start::before {
                        left: 50%;
                        border-radius: 100% 0 0 100%;
                    }
                    &.range-end::before {
                        left: 0;
                        border-radius: 0 100% 100% 0;
                    }
                    &.is-selected {
                        .button {
                            background: $primary;
                        }
                    }
                }
            }
        }

        .calendar-header {
            background: $primary;
            display: block;
            flex: none;
            .card-header-title {
                width: 100%;
                display: block;
                flex: none;
            }
            .icon {
                position: static;
                left: auto;
                top: auto;
                width: 3rem;
            }
            .level-year, .level-month {
                width: 100%;
                .button {
                    width: 100%;
                    border: none;
                    padding-top: 0.1rem;
                    padding-bottom: 0.1rem;
                    height: auto;
                    span {
                        text-shadow: 0 0 0.1rem adjust-color($primary, $lightness: -30%, $alpha: -0.8);
                    }
                }
                .level-item {
                    margin: 0;
                }
            }
            .level-year {
                margin-bottom: 0;
                .icon {
                    width: 3rem;
                }
            }
            .level-month {
                .title {
                    /*padding-bottom: 0.3rem;*/
                }
            }
        }
        .kydemy-month-picker, .kydemy-year-picker {
            .is-month, .is-year {
                .button {
                    width: 100%;
                }
            }
        }
        .kydemy-year-picker {
            .is-year {
                padding-top: 0.25rem;
                padding-bottom: 0.25rem;
            }
        }
        .kydemy-time-picker {
            .card {
                overflow: hidden;
                .card-content {
                    padding-top: 0.5rem;
                    padding-bottom: 0.5rem;
                }
            }
            .columns {
                .column {
                    .button {
                        width: 100%;
                    }
                    .separator {
                        display: block;
                        padding-top: calc(0.375em - 1px);
                        padding-bottom: calc(0.375em - 1px);
                        height: 2.25em;
                        line-height: 1.5;
                    }
                }
            }
            .kydemy-hour-picker, .kydemy-minute-picker {
                .columns .column {
                    padding-top: 0;
                    padding-bottom: 0;
                }
            }
            .kydemy-minute-picker {
                .column {
                    text-align: center;
                    flex: none;
                    width: 16.666666667%;
                }
            }
            .kydemy-hour-picker {
                .column {
                    text-align: center;
                    flex: none;
                }
                &.is-12-hour .column {
                    width: 16.666666667%;
                }
                &.is-24-hour .column {
                    width: 12.5%;
                }
            }
        }
    }

    .kydemy-date-picker, .kydemy-month-picker, .kydemy-year-picker, .kydemy-time-picker, .kydemy-hour-picker, .kydemy-minute-picker {
        .fade-leave-to {
            position: absolute;
            top: 0;
        }
        .card-content {
            background: white;
            overflow: hidden;
        }
    }
</style>