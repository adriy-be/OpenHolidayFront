<script>
export default {
  name: 'MonthCalendar',
  data() {
    return {
      weekdays: ['Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi', 'Dimanche'],
      days: [],
      monthStr: new Intl.DateTimeFormat('fr-FR', { month: 'long' }).format(
        new Date(this.year, this.month - 1),
      ), // Get the full month name
    }
  },
  created() {
    this.generateDays().then((days) => {
      this.days = days
    })
  },
  methods: {
    getDaysInMonth(year, month) {
      // Returns the number of days in a given month (1-based month index)
      return new Date(year, month, 0).getDate()
    },
    getFirstDayOffset(year, month) {
      // Returns the offset for the first day of the month (0 = Monday, 6 = Sunday)
      const firstDay = new Date(year, month - 1, 1).getDay()
      return firstDay === 0 ? 6 : firstDay - 1 // Adjust for Monday start
    },
    async generateDays() {
      const currentYear = this.year
      const currentMonth = this.month
      const daysInMonth = this.getDaysInMonth(currentYear, currentMonth)
      const firstDayOffset = this.getFirstDayOffset(currentYear, currentMonth)

      const days = Array(firstDayOffset).fill({ value: 0, holiday: 0 }) // Fill with empty slots for offset
      const holidays = await this.getMonthHolidays(currentYear, currentMonth, 'BE') // Fetch holidays for the month
      console.log('Holidays:', holidays)
      for (let i = 1; i <= daysInMonth; i++) {
        console.log('Current day:', i)
        console.log(
          holidays
            .filter((holiday) => {
              const holidayStart = new Date(holiday.dateStart)
              const holidayEnd = new Date(holiday.dateEnd)
              const currentDate = new Date(currentYear, currentMonth - 1, i)
              console.log('Current date:', currentDate)
              console.log('Holiday start:', holidayStart)
              console.log('Holiday end:', holidayEnd)
              console.log('Is holiday:', currentDate >= holidayStart && currentDate <= holidayEnd)
              console.log('Holiday type:', holiday.type)
              return currentDate >= holidayStart && currentDate <= holidayEnd
            })
            .map((holiday) => holiday.type),
        )
        days.push({
          value: i,
          daysOfWeek: new Date(currentYear, currentMonth - 1, i).getDay(),
          holiday: holidays
            .filter((holiday) => {
              const holidayStart = new Date(holiday.dateStart)
              const holidayEnd = new Date(holiday.dateEnd)
              const currentDate = new Date(currentYear, currentMonth - 1, i)
              return currentDate >= holidayStart && currentDate <= holidayEnd
            })
            .map((holiday) => holiday.type),
        })
      }
      console.log('Days:', days)
      return days
    },
    async getMonthHolidays(year, month, country) {
      let holidays = []
      await fetch(
        `/api/PublicHolidays?countryIsoCode=${country}&validFrom=${year}-${String(month).padStart(2, '0')}-01&validTo=${year}-${String(month).padStart(2, '0')}-${this.getDaysInMonth(year, month)}&languageIsoCode=FR&subdivisionCode=BE-FR`,
      )
        .then((response) => {
          if (!response.ok) {
            throw new Error('Failed to fetch holidays')
          }
          return response.json()
        })
        .then((data) => {
          console.log('Fetched school holidays:', data)
          holidays.push(
            ...data.map((holiday) => ({
              dateStart: new Date(holiday.startDate).setHours(0, 0, 0, 0),
              dateEnd: new Date(holiday.endDate).setHours(23, 59, 59, 999),
              type: 1,
              name: holiday.name.find((name) => name.language === 'EN')?.text || 'Unknown',
            })),
          )
        })
        .catch((error) => {
          console.error('Error fetching holidays:', error)
        })
      let startDate = new Date(year, month - 2, 0) // First day of the month
      let endDate = new Date(year, month, 0) // Last day of the month
      await fetch(
        `/api/SchoolHolidays?countryIsoCode=${country}&validFrom=${startDate.toISOString().split('T')[0]}&validTo=${endDate.toISOString().split('T')[0]}&languageIsoCode=FR&subdivisionCode=BE-FR`,
        //SchoolHolidays?countryIsoCode=DE&validFrom=2023-01-01&validTo=2023-12-31&languageIsoCode=DE&subdivisionCode=DE-MV
        //SchoolHolidays?countryIsoCode=BE&validFrom=2024-11-29&validTo=2025-01-30&languageIsoCode=FR&subdivisionCode=FR-BE
      )
        .then((response) => {
          if (!response.ok) {
            throw new Error('Failed to fetch holidays')
          }
          return response.json()
        })
        .then((data) => {
          console.log('Fetched holidays:', data)
          holidays.push(
            ...data.map((holiday) => ({
              dateStart: new Date(holiday.startDate).setHours(0, 0, 0, 0),
              dateEnd: new Date(holiday.endDate).setHours(23, 59, 59, 999),
              type: 2,
              name: holiday.name.find((name) => name.language === 'EN')?.text || 'Unknown',
            })),
          )
        })
        .catch((error) => {
          console.error('Error fetching holidays:', error)
        })
      return holidays
    },
  },
  props: {
    year: {
      type: Number,
      required: true,
    },
    month: {
      type: Number,
      required: true,
    },
  },
}
</script>
<template>
  <div class="month-name" style="text-align: center; padding-top: 10px">{{ monthStr }}</div>
  <div class="calendar-grid">
    <div class="weekday" v-for="day in weekdays">{{ day }}</div>
    <div
      class="day"
      v-for="day in days"
      :class="{
        holiday: Array.isArray(day.holiday) && day.holiday.includes(1),
        scolarHoliday: Array.isArray(day.holiday) && day.holiday.includes(2),
        empty: day.value === 0,
        weekend: day.daysOfWeek === 6 || day.daysOfWeek === 0,
      }"
    >
      {{ day.value !== 0 ? day.value : '' }}
    </div>
  </div>
</template>

<style scoped>
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr); /* 7 equal columns */
  box-sizing: border-box;
  padding: 8px;
}
.weekday,
.day {
  text-align: center;
  border: 1px solid #ccc;
  padding: 0.3em;
  margin: 0.3em;
  box-sizing: border-box;
  font-size: 0.55em;
  width: 6em;
}
.holiday {
  background-color: #5466cc81 !important;
}
.scolarHoliday {
  background-color: #38aa8e81;
}
.day.empty {
  background-color: transparent;
  border-color: transparent;
}
.weekend {
  background-color: #f0f0f0; /* Light gray for weekends */
}
</style>
