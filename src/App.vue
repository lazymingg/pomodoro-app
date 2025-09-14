  <script setup>
  import { ref, computed } from 'vue';
  import Timer from './components/timer.vue';
  import Todo from './components/todo.vue';
  
  const theme = [
    { name: 'Pomodoro', dataTheme: 'dark' },
    { name: 'Short Break', dataTheme: 'light' },
    { name: 'Long Break', dataTheme: 'light' },
  ];

  const currentOption = ref(theme[0].name);
  
  // Set initial theme based on default option (Pomodoro = dark)
  document.documentElement.setAttribute("data-theme", theme[0].dataTheme);
  
  function changeTheme(label) {
    const selected = theme.find((item) => item.name === label);
    if (selected) {
      currentOption.value = selected.name;
      document.documentElement.setAttribute('data-theme', selected.dataTheme);
    }
  }
</script>

  <template>
    <div class="body">
      <Timer @changeTimer="changeTheme" />
      <Todo />
    </div>
    <footer>
      <p>2025 a gift for thanhbinhhh :D.</p>
    </footer>
  </template>

<style scoped>
.body {
  background-color: var(--color-background);
  color: var(--color-text);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 2rem 1rem;
  gap: 2rem;
  transition: background-color 1s, color 1s;
}

/* Responsive font sizes */
html {
  font-size: 16px;
}

@media (max-width: 1199px) {
  html {
    font-size: 15px;
  }
}

@media (max-width: 991px) {
  html {
    font-size: 14px;
  }
}

@media (max-width: 767px) {
  html {
    font-size: 13px;
  }
}

@media (max-width: 575px) {
  html {
    font-size: 12px;
  }
}
</style>
