<template>
  <div id="app">
    <h1>Мой Календарь</h1>
    <hr>

    <Calendar 
      :lang="langCalendar"
      initialDate="2024-12-25" 
      @date-selected="handleDateSelect" 
    />
    
    <p>Выбранная дата: {{ selected }}</p>

    <button @click="changelangCalendar(langCalendar)">Сменить язык</button>
  </div>
</template>

<script>
import Calendar from '../components/Calendar.vue';

export default {
  name: 'App',
  components: {
    Calendar
  },
  data() {
    return {
      selected: null,
      langCalendar: 'ru'
    }
  },
  methods: {
    handleDateSelect(date) {
      // date здесь - это строка "YYYY-MM-DD" из компонента Calendar
      console.log('Дата выбрана:', date);
      this.selected = date;
    },
    changelangCalendar(lang) {
      console.log(this.langCalendar);
      if (lang === 'ru') {
        this.langCalendar = 'en';
      } else {
        this.langCalendar = 'ru';
      }
    }
  },
}
</script>