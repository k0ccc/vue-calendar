<template>
  <div class="calendar-wrapper">
    <div class="calendar-header">
      <button @click="prevMonth" class="nav-btn">&lt;</button>
      <span class="current-month-year">{{ currentMonthYear }}</span>
      <button @click="nextMonth" class="nav-btn">&gt;</button>
    </div>

    <div class="weekdays">
      <div v-for="day in localeWeekdays" :key="day" class="weekday">
        {{ day }}
      </div>
    </div>

    <div class="days-grid">
      <div
        v-for="padding in startDayOfWeek"
        :key="`padding-${padding}`"
        class="day-cell empty"
      ></div>

      <div
        v-for="day in daysInMonth"
        :key="day"
        class="day-cell"
        :class="{
          today: isToday(day),
          selected: isSelected(day),
        }"
        @click="selectDate(day)"
      >
        {{ day }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Calendar",
  props: {
    /**
     * Начальная дата для инициализации календаря.
     * Ожидаемый формат: "год-месяц-день" (YYYY-MM-DD).
     * 
     */
    initialDate: {
      type: String,
      default: null,
    },
    /**
     * Язык для отображения (ru/en).
     * Влияет на названия месяцев и дней недели.
     */
    lang: {
      type: String,
      default: "ru", // По умолчанию русский
    },
  },
  data() {
    return {
      // displayDate - дата, которая определяет отображаемый *месяц*
      displayDate: new Date(),
      // selectedDate - дата, которую выбрал пользователь
      selectedDate: null,
      // Локализация для языков
      locales: {
        ru: {
          months: [
            "Январь", "Февраль", "Март", "Апрель", "Май", "Июнь",
            "Июль", "Август", "Сентябрь", "Октябрь", "Ноябрь", "Декабрь"
          ],
          weekdays: ["Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс"],
        },
        en: {
          months: [
            "January", "February", "March", "April", "May", "June",
            "July", "August", "September", "October", "November", "December"
          ],
          weekdays: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
        },
      },
    };
  },
  computed: {
    /**
     * Вычисляет строку "Месяц Год" на основе displayDate и выбранного языка.
     */
    currentMonthYear() {
      if (!this.locales[this.lang]) return "";
      const monthName = this.locales[this.lang].months[this.displayDate.getMonth()];
      const year = this.displayDate.getFullYear();
      return `${monthName} ${year}`;
    },

    /**
     * Возвращает массив названий дней недели для текущего языка.
     */
    localeWeekdays() {
      return this.locales[this.lang]?.weekdays || this.locales.en.weekdays;
    },

    /**
     * Вычисляет количество дней в отображаемом месяце.
     */
    daysInMonth() {
      const year = this.displayDate.getFullYear();
      const month = this.displayDate.getMonth();
      // new Date(year, month + 1, 0) - трюк для получения последнего дня предыдущего месяца
      const lastDay = new Date(year, month + 1, 0).getDate();
      
      const daysArray = [];
      for (let day = 1; day <= lastDay; day++) {
        daysArray.push(day);
      }
      return daysArray;
    },

    /**
     * Вычисляет, с какого дня недели начинается месяц (0 - Пн, 6 - Вс).
     * Это нужно для "пустых" ячеек в сетке.
     */
    startDayOfWeek() {
      const year = this.displayDate.getFullYear();
      const month = this.displayDate.getMonth();
      // getDay() возвращает 0 для Вс, 1 для Пн. Нам нужно 0 для Пн, 6 для Вс.
      let firstDay = new Date(year, month, 1).getDay();
      return firstDay === 0 ? 6 : firstDay - 1; // Коррекция (Вс = 0 -> 6, Пн = 1 -> 0)
    }
  },
  methods: {
    /**
     * Переключение на предыдущий месяц. 
     */
    prevMonth() {
      this.displayDate = new Date(
        this.displayDate.getFullYear(),
        this.displayDate.getMonth() - 1,
        1
      );
    },
    
    /**
     * Переключение на следующий месяц. 
     */
    nextMonth() {
      this.displayDate = new Date(
        this.displayDate.getFullYear(),
        this.displayDate.getMonth() + 1,
        1
      );
    },

    /**
     * Выбор даты по клику. 
     * Отправка события 'date-selected' с датой в формате YYYY-MM-DD. 
     */
    selectDate(day) {
      this.selectedDate = new Date(
        this.displayDate.getFullYear(),
        this.displayDate.getMonth(),
        day
      );
      
      // Отправляем событие родителю
      this.$emit('date-selected', this.formatDateToYMD(this.selectedDate));
    },

    /**
     * Парсит строку "YYYY-MM-DD" в объект Date.
     */
    parseDateFromYMD(dateString) {
      const parts = dateString.split('-');
      if (parts.length === 3) {
        const year = parseInt(parts[0], 10);
        const month = parseInt(parts[1], 10) - 1; // Месяцы в JS с 0
        const day = parseInt(parts[2], 10);
        
        // Простая валидация
        const date = new Date(year, month, day);
        if (!isNaN(date.getTime()) && 
            date.getFullYear() === year &&
            date.getMonth() === month &&
            date.getDate() === day) {
          return date;
        }
      }
      return null;
    },

    /**
     * Форматирует объект Date в строку "YYYY-MM-DD".
     */
    formatDateToYMD(date) {
      if (!date || isNaN(date.getTime())) return null;

      const year = date.getFullYear();
      // Добавляем '0' спереди и берем 2 последних символа
      const month = ('0' + (date.getMonth() + 1)).slice(-2);
      const day = ('0' + date.getDate()).slice(-2);
      
      return `${year}-${month}-${day}`;
    },

    /**
     * Проверяет, является ли `day` сегодняшним днем.
     */
    isToday(day) {
      const today = new Date();
      return today.getDate() === day &&
             today.getMonth() === this.displayDate.getMonth() &&
             today.getFullYear() === this.displayDate.getFullYear();
    },

    /**
     * Проверяет, является ли `day` выбранной датой.
     */
    isSelected(day) {
      if (!this.selectedDate) return false;
      
      return this.selectedDate.getDate() === day &&
             this.selectedDate.getMonth() === this.displayDate.getMonth() &&
             this.selectedDate.getFullYear() === this.displayDate.getFullYear();
    }
  },
  /**
   * Инициализация при создании компонента.
   */
  created() {
    let dateToSet;
    
    // 4. Проверяем, передано ли свойство 'initialDate' 
    if (this.initialDate) {
      const parsedDate = this.parseDateFromYMD(this.initialDate);
      if (parsedDate) {
        dateToSet = parsedDate; // Используем переданную дату 
      } else {
        console.error("Calendar: Invalid initialDate format. Use 'YYYY-MM-DD'.");
        dateToSet = new Date(); // Фоллбэк на текущую, если формат неверный 
      }
    } else {
      // 5. Если дата не передана, берем текущий день 
      dateToSet = new Date();
    }
    
    // Устанавливаем и выбранную дату, и месяц для отображения
    this.selectedDate = dateToSet;
    // Устанавливаем displayDate на 1-е число того же месяца,
    // чтобы календарь открылся на этом месяце.
    this.displayDate = new Date(dateToSet.getFullYear(), dateToSet.getMonth(), 1);
  }
};
</script>

<style scoped>
.calendar-wrapper {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  max-width: 320px;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.current-month-year {
  font-size: 1.1rem;
  font-weight: 600;
  color: #333;
}

.nav-btn {
  background-color: transparent;
  border: 1px solid transparent;
  padding: 4px 8px;
  cursor: pointer;
  font-size: 1.2rem;
  color: #555;
  border-radius: 4px;
}
.nav-btn:hover {
  background-color: #f0f0f0;
}
.nav-btn:active {
  background-color: #e0e0e0;
}

.weekdays,
.days-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
}

.weekday {
  font-size: 0.8rem;
  font-weight: 500;
  color: #888;
  padding: 8px 0;
}

.day-cell {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
  border-radius: 50%; /* Круглая форма */
  cursor: pointer;
  font-size: 0.9rem;
  transition: background-color 0.2s, color 0.2s;
}

.day-cell:not(.empty):hover {
  background-color: #f0f0f0;
}

.day-cell.empty {
  cursor: default;
}

/* Стиль для сегодняшнего дня */
.day-cell.today {
  font-weight: bold;
  color: #007bff;
  border: 1px solid #007bff;
}

/* Стиль для выбранного дня */
.day-cell.selected {
  background-color: #007bff;
  color: #ffffff;
  border: 1px solid #007bff;
}
.day-cell.selected:hover {
  background-color: #0056b3;
}

.day-cell.today.selected {
  background-color: #007bff;
  color: #ffffff;
}

</style>