<template>
    <div class="parser">
      <!-- Ввод корневого селектора -->
      <label for="root-selector" class="parser__label">Root Selector:</label>
      <input
        id="root-selector"
        v-model="rootSelector"
        placeholder="#list-root"
        class="parser__field"
      />
  
      <!-- Ввод селектора элемента списка -->
      <label for="item-selector" class="parser__label">Item Selector:</label>
      <input
        id="item-selector"
        v-model="itemSelector"
        placeholder=".list-item"
        class="parser__field"
      />
  
      <!-- Ввод структуры объекта -->
      <label for="structure" class="parser__label">Structure (JSON):</label>
      <textarea
        id="structure"
        v-model="structureInput"
        placeholder='{ "name": ".item-title", "price": ".item-price" }'
        class="parser__field parser__field--textarea"
      ></textarea>
  
      <!-- Ввод HTML-кода -->
      <label for="html-input" class="parser__label">HTML Code:</label>
      <textarea
        id="html-input"
        v-model="htmlInput"
        placeholder="Paste your HTML here..."
        class="parser__field parser__field--textarea"
      ></textarea>
  
      <!-- Кнопка для запуска парсинга -->
      <button class="parser__button" @click="() => parseHtml()">
        Парсить HTML
      </button>
  
      <!-- Результат парсинга -->
      <div class="parser__result-container">
        <h3>Результат парсинга:</h3>
        <pre class="parser__result-array">{{ parsedData }}</pre>
      </div>
    </div>
</template>

<script>
export default {
  name: "ListParser",
  data() {
    return {
      rootSelector: "", // Ввод корневого селектора
      itemSelector: "", // Ввод селектора элемента списка
      structureInput: "", // Ввод структуры объекта (JSON в строке)
      htmlInput: "", // Ввод HTML-кода
      parsedData: [], // Результат парсинга
    };
  },
  methods: {
    parseHtml() {
      try {
        const structure = JSON.parse(this.structureInput);
        const container = document.createElement("div");
        container.innerHTML = this.htmlInput;

        const root = container.querySelector(this.rootSelector);
        if (!root) {
          console.error(`Root element "${this.rootSelector}" не найден.`);
          this.parsedData = [];
          return;
        }

        const items = root.querySelectorAll(this.itemSelector);
        if (!items.length) {
          console.error(
            `Элементы списка по селектору "${this.itemSelector}" не найдены.`
          );
          this.parsedData = [];
          return;
        }

        this.parsedData = Array.from(items).map((item) => {
          const parsedItem = {};

          for (const [key, selector] of Object.entries(structure)) {
            const fields = item.querySelectorAll(selector);

            // Если найдено несколько элементов, формируем массив значений
            if (fields.length > 1) {
              parsedItem[key] = Array.from(fields).map((field) =>
                this.cleanText(field.textContent)
              );
            } else {
              // Если найден один элемент или ничего
              parsedItem[key] = fields.length
                ? this.cleanText(fields[0].textContent)
                : null;
            }
          }

          return parsedItem;
        });
      } catch (error) {
        console.error("Ошибка парсинга JSON структуры:", error);
        this.parsedData = [];
      }
    },
    cleanText(text) {
      return text
        .replace(/\s+/g, " ") // Убираем лишние пробелы
        .trim(); // Убираем пробелы в начале и конце
    },
  },
};
</script>

<style scoped lang="less">
@import "styles/styles.less";
</style>