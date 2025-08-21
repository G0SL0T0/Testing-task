<template>
  <div class="calendar">
    <div class="calendar-header">
      <button @click="previousMonth">&lt;</button>
      <div class="month-year">{{ monthNames[currentMonth] }} {{ currentYear }}</div>
      <button @click="nextMonth">&gt;</button>
    </div>
    
    <div class="weekdays">
      <div 
        v-for="dayIndex in 7" 
        :key="dayIndex" 
        class="weekday"
      >
        {{ weekdayNames[dayIndex - 1] }}
      </div>
    </div>
    
    <div class="days">
      <div 
        v-for="dayIndex in totalDays" 
        :key="dayIndex"
        :class="['day', { 
          'other-month': !isCurrentMonth(dayIndex),
          'selected': isSelected(dayIndex),
          'today': isToday(dayIndex)
        }]"
        @click="selectDate(dayIndex)"
      >
        {{ getDayNumber(dayIndex) }}
      </div>
    </div>
    
    <div class="language-selector">
      <select v-model="selectedLanguage" @change="changeLanguage">
        <option value="en">English</option>
        <option value="ru">Русский</option>
      </select>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue';

export default {
  name: 'Calendar',
  props: {
    initialDate: {
      type: String,
      default: '2025-08-22'
    },
    language: {
      type: String,
      default: 'en'
    }
  },
  emits: ['date-selected'],
  setup(props, { emit }) {
    const currentDate = ref(new Date());
    const monthNames = ref([]);
    const weekdayNames = ref([]);
    const selectedDate = ref(null);
    const firstDayOffset = ref(0);
    const daysInCurrentMonth = ref(0);
    const daysInPrevMonth = ref(0);
    const selectedLanguage = ref(props.language);
    
    const currentYear = computed(() => currentDate.value.getFullYear());
    const currentMonth = computed(() => currentDate.value.getMonth());
    
    // Динамический расчет дней в календаре на основе недель
    const totalDays = computed(() => {
      const weeksNeeded = Math.ceil((firstDayOffset.value + daysInCurrentMonth.value) / 7);
      return weeksNeeded * 7;
    });
    
    // Генератор локализованных данных
    const loadLocaleData = () => {
      const localeGenerator = (lang) => {
        if (lang === 'ru') {
          return {
            months: ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 
                     'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'],
            weekdays: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
          };
        } else {
          return {
            months: ['January', 'February', 'March', 'April', 'May', 'June', 
                     'July', 'August', 'September', 'October', 'November', 'December'],
            weekdays: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
          };
        }
      };
      
      const locale = localeGenerator(selectedLanguage.value);
      monthNames.value = locale.months;
      weekdayNames.value = locale.weekdays;
    };
    
    // Обновление данных календаря с началом недели "понедельник"
    const updateCalendarData = () => {
      const year = currentYear.value;
      const month = currentMonth.value;
      
      const firstDay = new Date(year, month, 1);
      // Преобразование: Вс(0) -> 6, Пн(1) -> 0, ..., Сб(6) -> 5
      firstDayOffset.value = (firstDay.getDay() + 6) % 7;
      
      daysInCurrentMonth.value = new Date(year, month + 1, 0).getDate();
      daysInPrevMonth.value = new Date(year, month, 0).getDate();
    };
    
    // Проверка принадлежности дня к текущему месяцу
    const isCurrentMonth = (dayIndex) => {
      const start = firstDayOffset.value;
      const end = start + daysInCurrentMonth.value;
      return dayIndex >= start && dayIndex < end;
    };
    
    // Расчет номера дня для трех случаев: предыдущий, текущий, следующий месяцы
    const getDayNumber = (dayIndex) => {
      const start = firstDayOffset.value;
      
      if (dayIndex < start) {
        // Дни предыдущего месяца
        return daysInPrevMonth.value - (start - dayIndex - 1);
      } else if (dayIndex < start + daysInCurrentMonth.value) {
        // Дни текущего месяца
        return dayIndex - start + 1;
      } else {
        // Дни следующего месяца
        return dayIndex - (start + daysInCurrentMonth.value) + 1;
      }
    };
    
    // Получение объекта Date для указанного дня
    const getDateForDay = (dayIndex) => {
      const year = currentYear.value;
      const month = currentMonth.value;
      const dayNum = getDayNumber(dayIndex);
      
      let targetMonth = month;
      if (dayIndex < firstDayOffset.value) {
        targetMonth = month - 1;
      } else if (dayIndex >= firstDayOffset.value + daysInCurrentMonth.value) {
        targetMonth = month + 1;
      }
      
      return new Date(year, targetMonth, dayNum);
    };
    
    const previousMonth = () => {
      currentDate.value = new Date(currentYear.value, currentMonth.value - 1, 1);
    };
    
    const nextMonth = () => {
      currentDate.value = new Date(currentYear.value, currentMonth.value + 1, 1);
    };
    
    // Обработка выбора даты с выводом в консоль
    const selectDate = (dayIndex) => {
      const date = getDateForDay(dayIndex);
      selectedDate.value = date;
      
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      const formattedDate = `${year}-${month}-${day}`;
      
      console.log('Выбрана дата:', formattedDate);
      
      emit('date-selected', formattedDate);
    };
    
    // Проверка выбранной даты через временную метку
    const isSelected = (dayIndex) => {
      if (!selectedDate.value) return false;
      const date = getDateForDay(dayIndex);
      return date.getTime() === selectedDate.value.getTime();
    };
    
    // Проверка сегодняшней даты по компонентам
    const isToday = (dayIndex) => {
      const today = new Date();
      const date = getDateForDay(dayIndex);
      return today.getFullYear() === date.getFullYear() &&
             today.getMonth() === date.getMonth() &&
             today.getDate() === date.getDate();
    };
    
    const changeLanguage = () => {
      loadLocaleData();
    };
    
    watch(() => props.language, () => {
      selectedLanguage.value = props.language;
      loadLocaleData();
    });
    
    watch(currentDate, () => {
      updateCalendarData();
    });
    
    // Инициализация календаря при изменении начальной даты
    watch(() => props.initialDate, (newDate) => {
      if (newDate) {
        const [year, month, day] = newDate.split('-').map(Number);
        currentDate.value = new Date(year, month - 1, 1);
        selectedDate.value = new Date(year, month - 1, day);
      } else {
        const today = new Date();
        currentDate.value = new Date(today.getFullYear(), today.getMonth(), 1);
        selectedDate.value = today;
      }
    }, { immediate: true });
    
    onMounted(() => {
      loadLocaleData();
      updateCalendarData();
    });
    
    return {
      currentYear,
      currentMonth,
      monthNames,
      weekdayNames,
      totalDays,
      previousMonth,
      nextMonth,
      selectDate,
      getDayNumber,
      isCurrentMonth,
      isSelected,
      isToday,
      selectedLanguage,
      changeLanguage
    };
  }
};
</script>

<style scoped>
.calendar {
  width: 300px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: Arial, sans-serif;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ddd;
}

.month-year {
  font-weight: bold;
}

button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  padding: 0 10px;
}

.weekdays {
  display: flex;
  justify-content: space-around;
  padding: 5px 0;
  border-bottom: 1px solid #eee;
}

.weekday {
  width: 14.28%;
  text-align: center;
  font-weight: bold;
  font-size: 12px;
}

.days {
  display: flex;
  flex-wrap: wrap;
}

.day {
  width: 14.28%;
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-sizing: border-box;
}

.day:hover {
  background-color: #f0f0f0;
}

.other-month {
  color: #ccc;
}

.selected {
  background-color: #4a90e2;
  color: white;
}

.today {
  font-weight: bold;
  color: #e74c3c;
}

.language-selector {
  padding: 10px;
  text-align: center;
}

.language-selector select {
  padding: 5px;
  border-radius: 4px;
  border: 1px solid #ddd;
}
</style>