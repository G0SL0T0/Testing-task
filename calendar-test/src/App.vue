<template>
  <div class="app">
    <h1>Календарь</h1>
    <div class="calendar-wrapper">
      <Calendar 
        :initial-date="initialDate" 
        :language="language" 
        @date-selected="handleDateSelected"
      />
    </div>
    
    <div class="info-panel">
      <div class="controls">
        <label>
          Начальная дата:
          <input type="date" v-model="initialDate" />
        </label>
        
        <label>
          Язык:
          <select v-model="language">
            <option value="en">English</option>
            <option value="ru">Русский</option>
          </select>
        </label>
      </div>
      
      <div v-if="selectedDate" class="selected-date">
        Выбранная дата: {{ formatDateDisplay(selectedDate) }}
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
import Calendar from './components/Calendar.vue';

export default {
  name: 'App',
  components: {
    Calendar
  },
  setup() {
    const initialDate = ref('2025-08-22');
    const language = ref('ru');
    const selectedDate = ref(null);
    
    // Обработка выбранной даты с преобразованием и выводом в консоль
    const handleDateSelected = (date) => {
      const dateObj = new Date(date);
      const transformedDate = `${dateObj.getFullYear()}-${String(dateObj.getMonth() + 1).padStart(2, '0')}-${String(dateObj.getDate()).padStart(2, '0')}`;
      selectedDate.value = transformedDate;
      console.log('Выбрана дата:', transformedDate);
    };
    
    // Форматирование даты в нестандартный вид "Д ДД.М ММ.Г ГГГГ"
    const formatDateDisplay = (dateStr) => {
      const date = new Date(dateStr);
      const day = date.getDate();
      const month = date.getMonth() + 1;
      const year = date.getFullYear();
      return `Д ${day}.М ${month}.Г ${year}`;
    };
    
    // Наблюдатель за изменением начальной даты
    watch(initialDate, (newVal) => {
      console.log('Начальная дата изменена:', newVal);
      const date = new Date(newVal);
      console.log('Преобразованная дата:', date.toLocaleDateString());
    });
    
    // Наблюдатель за изменением языка
    watch(language, (newVal) => {
      console.log('Язык изменен:', newVal);
      const messages = {
        en: 'Language changed to English',
        ru: 'Язык изменен на русский'
      };
      console.log(messages[newVal] || 'Language changed');
    });
    
    return {
      initialDate,
      language,
      selectedDate,
      handleDateSelected,
      formatDateDisplay
    };
  }
};
</script>

<style>
.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.calendar-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.info-panel {
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.controls {
  display: flex;
  gap: 20px;
  margin-bottom: 15px;
}

.controls label {
  display: flex;
  align-items: center;
  gap: 8px;
}

.controls input, .controls select {
  padding: 5px;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.selected-date {
  padding: 10px;
  background-color: #e8f4fd;
  border-left: 4px solid #4a90e2;
  border-radius: 4px;
}
</style>