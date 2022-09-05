<template>
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title class="text-center"> {{title}} </q-toolbar-title>
      </q-toolbar>
    </q-header>
  <q-page class="flex items-center column q-mt-md">
  <div class="flex flex-center column q-pa-xs" style="border: 2pt solid grey;">
    <div style="font-size: 1.1em;"><b>{{metricsLbl}}</b></div>
    <div class="flex row">
      <div class="flex column flex-center q-pa-sm q-ma-xs" style="border: 2pt solid grey; font-size: 1.2em">
        <div>{{parityErrorLbl}}</div>
        <div><b>{{parityError}}</b></div>
      </div>
      <div class="flex column flex-center q-pa-sm q-ma-xs" style="border: 2pt solid grey; font-size: 1.2em">
        <div>{{weekdayErrorLbl}}</div>
        <div><b>{{weekdayError}}</b></div>
      </div>
      <div class="flex column flex-center q-pa-sm q-ma-xs" style="border: 2pt solid grey; font-size: 1.2em">
        <div>{{partErrorLbl}}</div>
        <div><b>{{partError}}</b></div>
      </div>
    </div>
  </div>
  <div class="q-pa-sm q-ma-sm flex column" style="border: 2pt solid;" :style="{'border-color': isValid?'grey':'red'}" >
    <div>
      {{parametersLbl}}
      <span v-if="!isValid" class="text-negative"><b>({{isInvalidLbl}})</b></span>
    </div>
    <div class="flex row" style="" >
      <q-input class="q-ma-xs" outlined rounded v-model="startDate" readonly :label="semStartDateLbl">
        <template v-slot:append>
          <q-icon name="event" class="cursor-pointer">
            <q-popup-proxy cover transition-show="scale" transition-hide="scale">
              <q-date
                v-model="startDate"
                mask="YYYY-MM-DD"
                minimal
                :options="date => [1].includes(new Date(date).getDay())"
                first-day-of-week="1"
              >
                <div class="row items-center justify-end">
                  <q-btn v-close-popup label="Close" color="primary" flat />
                </div>
              </q-date>
            </q-popup-proxy>
          </q-icon>
        </template>
      </q-input>
      <q-input class="q-ma-xs" type="number" outlined rounded v-model.number="t" 
        :label="numOfWeeks" 
      />
      <q-input class="q-ma-xs" type="number" outlined rounded v-model.number="t1" 
        :label="numOfWeeksInFirstPart"
      />
      <div class="flex-center flex q-ma-xs">
      PL<q-toggle
        v-model="english"
        :color="english ? 'red' : 'blue'"
        :class="{'toggle-red' : english, 'toggle-blue': !english}"
        keep-color
      />EN</div>
    </div>
    </div>
    <div class="q-gutter-xs row bg-cyan-1 rounded-borders q-pa-xs q-mx-sm" style="max-width: 1024px; min-height:73px; border: 2pt solid #33a;">
    <q-badge color="red">{{holidaysBadgeLbl}}:</q-badge>
    <div>
    <q-chip
        v-for="(day, dayIdx) in holidaysInCalendar" :key="day" 
        removable
        color="primary"
        text-color="white"
        icon="free_cancellation"
        :label="holidaysInCalendar[dayIdx]"
        :title="holidaysInCalendar[dayIdx]"
        @remove="removeHoliday(holidaysInCalendar[dayIdx])"
      />
    </div>
    </div>
    <!--div v-for="(day, dayIdx) in holidays" :key="day" class="q-pa-md" style="max-width: 300px" >
      <q-input filled v-model="holidays[dayIdx]" readonly>
        <template v-slot:append>
          <q-icon name="event" class="cursor-pointer">
            <q-popup-proxy cover transition-show="scale" transition-hide="scale">
              <q-date
                v-model="holidays[dayIdx]"
                mask="YYYY-MM-DD"
                minimal
                :options="date => [1, 2, 3, 4, 5].includes(new Date(date).getDay())"
                first-day-of-week="1"
              >
                <div class="row items-center justify-end">
                  <q-btn v-close-popup label="Close" color="primary" flat />
                </div>
              </q-date>
            </q-popup-proxy>
          </q-icon>
        </template>
      </q-input>
    </div-->
    <!--q-btn label="Add" @click="addHoliday" /-->

    <div v-if="isValid" class="flex row">
    <table style="margin-bottom: 40pt;" class="q-ma-md">
    <tr>
        <td style="border: 2px solid black; width: 40pt; "></td>
        <td style="border: 2px solid black; width: 100pt; ">{{weekdays[0]}}</td>
        <td style="border: 2px solid black; width: 100pt; ">{{weekdays[1]}}</td>
        <td style="border: 2px solid black; width: 100pt; ">{{weekdays[2]}}</td>
        <td style="border: 2px solid black; width: 100pt; ">{{weekdays[3]}}</td>
        <td style="border: 2px solid black; width: 100pt; ">{{weekdays[4]}}</td>
    </tr>
    <tr v-for="(row, rowIdx) in betterCalendarMatrix" :key="row">
      <td style="border: 2px solid black; text-align: right">{{english?'w':'t'}}{{rowIdx+1}}/x{{(rowIdx % 2)+1}}</td>
      <td 
        @click="addHoliday(col, rowIdx, colIdx)" 
        v-for="(col, colIdx) in row" 
        :key="col" style="text-align: right" 
        :style="{'background-color': getBckColor(col, rowIdx, colIdx)}"
        class="cursor-pointer">
        <div v-if="col['free'] != true" :class="{'text-bold': col['part'] == 1}"><span v-html="getLabel(col, rowIdx, colIdx)"></span></div>
      </td>
    </tr>
    </table>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "PageIndex",

  data() {
    return {
      startDate: '2021-03-01',
      holidays: ['2021-01-01','2021-01-06','2021-04-04','2021-04-05','2021-05-01','2021-05-03','2021-05-23','2021-06-03','2021-08-15','2021-11-01','2021-11-11','2021-12-25','2021-12-26',
                 '2022-01-01','2022-01-06','2022-04-17','2022-04-18','2022-05-01','2022-05-03','2022-06-05','2022-06-16','2022-08-15','2022-11-01','2022-11-11','2022-12-25','2022-12-26',
                 '2023-01-01','2023-01-06','2023-04-09','2023-04-10','2023-05-01','2023-05-03','2023-05-28','2023-06-08','2023-08-15','2023-11-01','2023-11-11','2023-12-25','2023-12-26'],
      t: 15,
      t1: 8,
      english: true
    }
  },

  methods: {
    d: function (i) {
      return Math.min(5, 5*(this.t - i) + this.f)
    },

    getBckColor: function (el, ridx, cidx) {
      if (el["free"]) {
        return 'white'
      // } else if (el["weekday"] != cidx) {
      //   return 'yellow'
      } else if (el['parity']) {
        return '#edfaff'
      } else {
        return '#fff4f4'
      }
    },

    getLabel: function (col, ridx, cidx) {

      let getLbl = (col) => {
        return `${String(col['date'].getDate()).padStart(2, '0')}.${String(col['date'].getMonth()+1).padStart(2, '0')}`
      }
      let lbl = getLbl(col)

      if (col["free"]) {
        return ''
      } else if (col["weekday"] != cidx) {
        let lbl2 = getLbl(col['old'])
        let weekday = col['weekday']
        return `<b style="color: red;">(${this.weekdays[weekday]})</b>(${lbl}) ${lbl2}`
      } else if (col["week"] != ridx) {
        let lbl2 = getLbl(col['old'])
        return `(${lbl}) ${lbl2}`
      } else {
        return `${lbl}`
      }
    },

    dateToCell: function(startDate, date, matrix) {
      var diff = (new Date(date).getTime() - new Date(startDate).getTime()) / (1000 * 3600 * 24)
      let i = Math.floor(diff / 7)
      let j = diff % 7
      if (matrix == undefined || 
          i >= matrix.length || 
          matrix[i] == undefined || 
          j >= matrix[i].length ||
          j > 4) {
        return undefined
      }
      
      return matrix[i][j]
    },

    dateToString: function(date) {
      return `${date.getYear() + 1900}-${String(date.getMonth()+1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
    },

    addHoliday: function(col, rowIdx, colIdx) {
      let dateStr = `${this.dateToString(col['old']?col['old']['date']:col['date'])}`
      if (this.holidays.includes(dateStr)) {
        this.removeHoliday(dateStr)
      } else {
        this.holidays = this.holidays.concat([dateStr])
      }
    },

    removeHoliday: function(dateStr) {
      this.holidays = this.holidays.filter((el) => { return el != dateStr })
    },

    countDays: function(prop, val) {
      let sum = 0
      for (let row of this.betterCalendarMatrix) {
        for (let cell of row) {
          if (cell[prop] == val && cell['free'] != true) {
            sum += 1
          }
        }
      }
      return sum
    },

    getError: function(prop) {
      let sum = 0
      for (let i = 0; i < this.betterCalendarMatrix.length; i++) {
        let row = this.betterCalendarMatrix[i]
        for (let j = 0; j < row.length; j++) {
          let cell1 = this.betterCalendarMatrix[i][j]
          let cell2 = this.calendarMatrix[i][j]
          if (cell1[prop] != cell2[prop]) {
            sum += 1
          }
        }
      }
      return sum
    },

    getHolidaysInCalendar: function() {
      let sum = 0
      let prev = 0
      let endDate = undefined
      
      do {
        endDate = new Date(this.startDate)
        endDate.setDate(endDate.getDate()+this.t*7 + sum + Math.floor(sum/2)*2 )
        prev = sum
        sum = this.holidays
        .filter( (el) => {
          return new Date(el) >= new Date(this.startDate) 
              && new Date(el) <= endDate
              && [1, 2, 3, 4, 5].includes((new Date(el)).getDay())
        }).length
      } while (sum != prev)

      return this.holidays
      .filter( (el) => {
        return new Date(el) >= new Date(this.startDate) 
            && new Date(el) <= endDate
            && [1, 2, 3, 4, 5].includes((new Date(el)).getDay())
      }).sort()
      
    }

  },

  computed: {

    isValid: function() {
      if (
        this.t < 2
        || this.t1 < 1
        || this.t1 > this.t
        || this.totalWeeksForEachWeekday
        .map((v, idx) => v - this.part1totalWeeksForEachWeekday[idx])
        .filter((v) => v < 0)
        .length > 0
      ) return false


      return true
    },

    totalWeeksForEachWeekday: function() {
      let sum = []
      for (let i = 0; i < 5; i++) {
        sum[i] = this.w - 1 + (this.d(this.w-1) > i ? 1 : 0)
      }
      
      return sum
    },

    part1totalWeeksForEachWeekday: function(prop, val) {
      let sum = [0,0,0,0,0]
      for (let i = 0; i < 5; i++) {
        for (let row of this.betterCalendarMatrix) {
          if (row.length > i && (row[i]['part'] == 1 || row[i]['free'])) {
            sum[i] += 1
          }
        }
      }

      return sum
    },

    parityError: function() {
      return this.getError('parity')
    },

    weekdayError: function() {
      return this.getError('weekday')
    },

    partError: function() {
      return this.getError('part')
    },

    weekdays: function() {
      if (this.english) {
        return ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
      } else {
        return ['Pn', 'Wt', 'Śr', 'Cz', 'Pt', 'So', 'Nd']
      }
    },

    semStartDateLbl: function() {
      return this.english ? 'Semester start date' : 'Data rozpoczęcia semestru'
    },

    numOfWeeks: function() {
      return this.english ? 'Number of weeks' : 'Liczba tygodni'
    },

    numOfWeeksInFirstPart: function() {
      return this.english ? 'Number of weeks in first part' : 'Liczba tyg. w pierwszej części sem.'
    },

    holidaysBadgeLbl: function() {
      return this.english ? 'Holidays (click on calendar day to add)' : 'Dni wolne (kliknij dzień aby dodać)'
    },

    title: function() {
      return this.english ? 'Academic calendar scheduler' : 'Generator kalendarza semestralnego'
    },

    parametersLbl: function() {
      return this.english ? 'Parameters' : 'Parametry'
    },

    isInvalidLbl: function() {
      return this.english ? 
        'Parameters are invalid. Please make sure the number of weeks, number of weeks in first part and number of holidays are valid.' : 
        'Parametry są niepoprawne. Upewnij się, że liczba tygodni w obu częściach semestru oraz liczba dni wolnych są poprawne.'
    },

    metricsLbl: function() {
      return this.english ? 'Metrics' : 'Metryki'
    },

    parityErrorLbl: function() {
      return this.english ? 'Parity error' : 'Błąd parzystości tyg.'
    },

    weekdayErrorLbl: function() {
      return this.english ? 'Weekday error' : 'Błąd dnia tyg.'
    },

    partErrorLbl: function() {
      return this.english ? 'Part error' : 'Błąd połowy sem.'
    },

    F: function() {
      return this.holidays
        .map((el) => { return this.dateToCell(this.startDate, el, this.calendarMatrix)} )
        .filter((el) => el != undefined)
    },

    holidaysInCalendar: function() {
      return this.getHolidaysInCalendar()
    },

    f: function() {
      return this.holidaysInCalendar.length
    },

    w: function() {
      return this.t + Math.ceil(this.f / 5)
    },

    endDate: function() {
      let sd = new Date(this.startDate)
      sd.setDate(sd.getDate()+this.t*7)
      return sd
    },

    betterCalendarT1: function() {
      return this.countDays('part', 1)
    },

    betterCalendarT2: function() {
      return this.countDays('part', 2)
    },

    betterCalendarParity1: function() {
      return this.countDays('parity', 0)
    },

    betterCalendarParity2: function() {
      return this.countDays('parity', 1)
    },

    calendarMatrix: function() {
      let arr = []
      let sd = new Date(this.startDate)
      for(let i = 0; i < this.w; i++) {
        let row = []
        for(let j = 0; j < this.d(i); j++) {
          let el = {
            'date': new Date(sd),//sd.getDate()+i*7 + j,
            'week': i,
            'weekday': j,
            'part': i < this.t1 ? 1 : 2,
            'parity': (i + 1) % 2,
            'free': false
          }
          row.push(el)
          sd.setDate(sd.getDate()+1)
        }
        arr.push(row);
        sd.setDate(sd.getDate()+2)
      }
      return arr
    },

    betterCalendarMatrix: function() {
      let freeDays = []
      let C = this.calendarMatrix.map((row) => {
        // return el
        return row.map(el => {
          if (this.F.includes(el)) {
            freeDays.push({...el})
            Object.assign(el, {'free': true})
          }
          return {...el}
        })
      })

      let fill_spare_days = (cmp) => {
        for (let i = 0; i < this.A.length; i++) {
          let a1 = this.A[i]
          let c1 = C[a1.week][a1.weekday]
          if (c1['assigned'] == undefined) {
            for (let j = 0; j < this.F.length; j++) {
              let a2 = this.F[j]
              let c2 = C[a2.week][a2.weekday]
              if (c2['assigned'] == undefined) {
                if (cmp(c1, c2)) {
                  let c1bck = {...c1}
                  Object.assign(c1, {
                    'old': c1bck, 
                    'parity': c2['parity'],
                    'weekday': c2['weekday'],
                    'week': c2['week'],
                    'date': c2['date'],
                    'assigned': true
                  })
                  Object.assign(c2, {'assigned': true})
                  break;
                }
              }
            }
          }
        }
      }

      fill_spare_days((c1, c2) => { 
        return c2['parity'] == c1['parity'] && c2['weekday'] == c1['weekday']
      })

      fill_spare_days((c1, c2) => { 
        return c2['weekday'] == c1['weekday']
      })

      fill_spare_days((c1, c2) => { 
        return c2['parity'] == c1['parity']
      })

      fill_spare_days((c1, c2) => { 
        return true
      })

      // # Step 4 - Refill part one with lacking days
      // Tt2 = filter(lambda x: x['part'] == 1, T)
      // for el in Tt2:
      //     for row in C:
      //         if row[el['weekday']]['part'] == 2 and not row[el['weekday']]['free']:
      //             print(row[el['weekday']], 'to first')
      //             row[el['weekday']]['part'] = 1
      //             break
      
      let part1elements = this.F.filter((el) => {return el['part'] == 1})
      for (let el of part1elements) {
        let spareInPart2 = C.flat().filter( 
          (x) => x['weekday'] == el['weekday'] && x['parity'] == el['parity'] && x['part'] == 2 && !x['free']
        )
        if (spareInPart2.length <= 0) {
          //error (should never happen)
          return []
        }
        spareInPart2[0]['part'] = 1
      }

      return C
    },

    A: function() {
      let arr = []
      for (let i = this.t; i < this.w; i++) {
        for (let j = 0; j < this.d(i); j++) {
          arr.push(this.calendarMatrix[i][j])
        }
      }
      return arr
    }


  },

  created() {
    let eng = localStorage.getItem('english') ?? 'true'
    this.english = eng == 'true'

    let tt = localStorage.getItem('t')
    this.t = tt == undefined ? 15 : parseInt(tt)

    let t1t = localStorage.getItem('t1')
    this.t1 = t1t == undefined ? 8 : parseInt(t1t)

    let startDatet = localStorage.getItem('startDate')
    this.startDate = startDatet == undefined ? '2021-03-01' : startDatet

    let holidayst = localStorage.getItem('holidays')
    if (holidayst != undefined) {
      Object.assign(this.holidays, JSON.parse(holidayst))
    }
  },

  watch: {
    english: function (val) {
      localStorage.setItem('english', val);
    },

    t: function (val) {
      localStorage.setItem('t', val);
    },

    t1: function (val) {
      localStorage.setItem('t1', val);
    },

    startDate: function (val) {
      localStorage.setItem('startDate', val);
    },

    holidays: function (val) {
      localStorage.setItem('holidays', JSON.stringify(val));
    },

    
  }

});
</script>
<style>
.toggle-red .q-toggle__thumb { border-color: red; }
.toggle-red .q-toggle__thumb:after { background-color: red; }

.toggle-blue .q-toggle__thumb { border-color: blue; }
.toggle-blue .q-toggle__thumb:after { background-color: blue; }
</style>