<!-- todo component -->
<script setup>
import { ref, computed, onMounted, watch } from 'vue';

// Reactive data
const newTaskText = ref('');
const editingTaskId = ref(null);
const editingText = ref('');

// Categories with tasks
const categories = ref([
  { name: 'Not Started', tasks: [], color: '#ff6b6b' },
  { name: 'In Progress', tasks: [], color: '#4ecdc4' },
  { name: 'Completed', tasks: [], color: '#45b7d1' }
]);

// Load data from localStorage on component mount
onMounted(() => {
  loadTasks();
});

// Watch for changes and save to localStorage
watch(categories, () => {
  saveTasks();
}, { deep: true });

// Functions
function loadTasks() {
  const saved = localStorage.getItem('pomodoro-tasks');
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      categories.value = parsed;
    } catch (error) {
      console.error('Error loading tasks:', error);
    }
  }
}

function saveTasks() {
  localStorage.setItem('pomodoro-tasks', JSON.stringify(categories.value));
}

function addTask() {
  if (newTaskText.value.trim()) {
    const newTask = {
      id: Date.now(),
      text: newTaskText.value.trim(),
      createdAt: new Date().toISOString()
    };
    categories.value[0].tasks.push(newTask);
    newTaskText.value = '';
  }
}

function deleteTask(categoryIndex, taskIndex) {
  categories.value[categoryIndex].tasks.splice(taskIndex, 1);
}

function startEditing(task) {
  editingTaskId.value = task.id;
  editingText.value = task.text;
}

function saveEdit(categoryIndex, taskIndex) {
  if (editingText.value.trim()) {
    categories.value[categoryIndex].tasks[taskIndex].text = editingText.value.trim();
  }
  cancelEdit();
}

function cancelEdit() {
  editingTaskId.value = null;
  editingText.value = '';
}

function moveTask(fromCategoryIndex, taskIndex, direction) {
  const task = categories.value[fromCategoryIndex].tasks[taskIndex];
  const toCategoryIndex = fromCategoryIndex + direction;
  
  if (toCategoryIndex >= 0 && toCategoryIndex < categories.value.length) {
    // Remove from current category
    categories.value[fromCategoryIndex].tasks.splice(taskIndex, 1);
    // Add to new category
    categories.value[toCategoryIndex].tasks.push(task);
  }
}

// Computed
const totalTasks = computed(() => {
  return categories.value.reduce((total, category) => total + category.tasks.length, 0);
});

const completedTasks = computed(() => {
  return categories.value[2].tasks.length;
});

const progress = computed(() => {
  if (totalTasks.value === 0) return 0;
  return Math.round((completedTasks.value / totalTasks.value) * 100);
});
</script>

<template>
  <div class="todo-container">
    <div class="todo-header">
      <h2>Tasks</h2>
      <div class="progress-info">
        <span>{{ completedTasks }}/{{ totalTasks }} completed ({{ progress }}%)</span>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progress + '%' }"></div>
        </div>
      </div>
    </div>

    <!-- Add new task -->
    <div class="add-task">
      <input
        v-model="newTaskText"
        @keyup.enter="addTask"
        placeholder="Add a new task..."
        class="task-input"
      />
      <button @click="addTask" class="add-btn">Add</button>
    </div>

    <!-- Task categories -->
    <div class="categories">
      <div
        v-for="(category, categoryIndex) in categories"
        :key="category.name"
        class="category"
      >
        <div class="category-header" :style="{ borderColor: category.color }">
          <h3>{{ category.name }}</h3>
          <span class="task-count">{{ category.tasks.length }}</span>
        </div>

        <div class="tasks">
          <div
            v-for="(task, taskIndex) in category.tasks"
            :key="task.id"
            class="task"
          >
            <!-- Task display mode -->
            <div v-if="editingTaskId !== task.id" class="task-content">
              <span class="task-text">{{ task.text }}</span>
              <div class="task-actions">
                <!-- Move buttons -->
                <button
                  v-if="categoryIndex > 0"
                  @click="moveTask(categoryIndex, taskIndex, -1)"
                  class="move-btn prev"
                  title="Move to previous stage"
                >
                  ←
                </button>
                <button
                  v-if="categoryIndex < categories.length - 1"
                  @click="moveTask(categoryIndex, taskIndex, 1)"
                  class="move-btn next"
                  title="Move to next stage"
                >
                  →
                </button>
                <!-- Edit and delete buttons -->
                <button
                  @click="startEditing(task)"
                  class="edit-btn"
                  title="Edit task"
                >
                  ✏️
                </button>
                <button
                  @click="deleteTask(categoryIndex, taskIndex)"
                  class="delete-btn"
                  title="Delete task"
                >
                  🗑️
                </button>
              </div>
            </div>

            <!-- Task edit mode -->
            <div v-else class="task-edit">
              <input
                v-model="editingText"
                @keyup.enter="saveEdit(categoryIndex, taskIndex)"
                @keyup.esc="cancelEdit"
                class="edit-input"
                ref="editInput"
              />
              <div class="edit-actions">
                <button @click="saveEdit(categoryIndex, taskIndex)" class="save-btn">Save</button>
                <button @click="cancelEdit" class="cancel-btn">Cancel</button>
              </div>
            </div>
          </div>

          <div v-if="category.tasks.length === 0" class="empty-category">
            No tasks yet
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.todo-container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  background-color: var(--color-background-soft);
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.todo-header {
  text-align: center;
  margin-bottom: 2rem;
}

.todo-header h2 {
  color: var(--color-heading);
  margin-bottom: 1rem;
  font-size: 2rem;
}

.progress-info {
  color: var(--color-text);
  font-size: 1rem;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background-color: var(--color-background-mute);
  border-radius: 4px;
  margin-top: 0.5rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ff6b6b, #4ecdc4, #45b7d1);
  transition: width 0.3s ease;
}

.add-task {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.task-input {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 2px solid var(--color-border);
  border-radius: 8px;
  background-color: var(--color-background);
  color: var(--color-text);
  font-size: 1rem;
}

.task-input:focus {
  outline: none;
  border-color: var(--color-border-hover);
}

.add-btn {
  padding: 0.75rem 1.5rem;
  background-color: var(--color-border);
  color: var(--color-heading);
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.add-btn:hover {
  background-color: var(--color-border-hover);
}

.categories {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.category {
  background-color: var(--color-background);
  border-radius: 8px;
  overflow: hidden;
  border: 2px solid var(--color-border);
}

.category-header {
  padding: 1rem;
  background-color: var(--color-background-mute);
  border-bottom: 3px solid;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.category-header h3 {
  color: var(--color-heading);
  margin: 0;
  font-size: 1.2rem;
}

.task-count {
  background-color: var(--color-border);
  color: var(--color-heading);
  padding: 0.25rem 0.5rem;
  border-radius: 12px;
  font-size: 0.875rem;
  font-weight: 600;
}

.tasks {
  padding: 1rem;
  min-height: 200px;
}

.task {
  background-color: var(--color-background-soft);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  margin-bottom: 0.75rem;
  padding: 0.75rem;
  transition: box-shadow 0.2s ease;
}

.task:hover {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.task-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.task-text {
  flex: 1;
  color: var(--color-text);
  word-break: break-word;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.move-btn, .edit-btn, .delete-btn {
  padding: 0.25rem 0.5rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.2s ease;
}

.move-btn {
  background-color: var(--color-border);
  color: var(--color-heading);
}

.move-btn:hover {
  background-color: var(--color-border-hover);
  transform: translateX(2px);
}

.move-btn.prev:hover {
  transform: translateX(-2px);
}

.edit-btn {
  background-color: transparent;
}

.edit-btn:hover {
  background-color: var(--color-background-mute);
}

.delete-btn {
  background-color: transparent;
}

.delete-btn:hover {
  background-color: #ff6b6b;
}

.task-edit {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.edit-input {
  padding: 0.5rem;
  border: 2px solid var(--color-border);
  border-radius: 4px;
  background-color: var(--color-background);
  color: var(--color-text);
  font-size: 1rem;
}

.edit-input:focus {
  outline: none;
  border-color: var(--color-border-hover);
}

.edit-actions {
  display: flex;
  gap: 0.5rem;
}

.save-btn, .cancel-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  font-weight: 600;
}

.save-btn {
  background-color: #4ecdc4;
  color: white;
}

.cancel-btn {
  background-color: var(--color-background-mute);
  color: var(--color-text);
}

.empty-category {
  text-align: center;
  color: var(--color-text);
  opacity: 0.6;
  font-style: italic;
  padding: 2rem;
}

@media (max-width: 768px) {
  .todo-container {
    padding: 1rem;
  }
  
  .categories {
    grid-template-columns: 1fr;
  }
  
  .add-task {
    flex-direction: column;
  }
  
  .task-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  
  .task-actions {
    width: 100%;
    justify-content: flex-end;
  }
}
</style>
