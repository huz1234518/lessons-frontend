
<script setup>
import { ref, onMounted } from 'vue';

// Store lessons retrieved from MongoDB
const lessons = ref([]);

// Track loading and error states
const loading = ref(true);
const errorMessage = ref('');

// Retrieve lessons from our Express.js backend
async function fetchLessons() {
  try {
    const response = await fetch('http://localhost:3000/lessons');

    if (!response.ok) {
      throw new Error('Failed to retrieve lessons');
    }

    lessons.value = await response.json();

    console.log('Lessons retrieved:', lessons.value);
  } catch (error) {
    console.error('Error fetching lessons:', error);
    errorMessage.value = 'Unable to load lessons. Please try again.';
  } finally {
    loading.value = false;
  }
}

// Load lessons when the page opens
onMounted(fetchLessons);
</script>

<template>
  <main class="container">
    <header>
      <h1>After-School Lessons</h1>
      <p>Discover and book exciting classes and activities!</p>
    </header>

    <!-- Loading message -->
    <p v-if="loading">Loading lessons...</p>

    <!-- Error message -->
    <p v-else-if="errorMessage" class="error">
      {{ errorMessage }}
    </p>

    <!-- Available lessons -->
    <section v-else>
      <h2>Available Lessons</h2>

      <p v-if="lessons.length === 0">
        No lessons are currently available.
      </p>

      <div v-else class="lesson-grid">
        <div
          v-for="lesson in lessons"
          :key="lesson._id"
          class="lesson-card"
        >
          <h3>{{ lesson.topic }}</h3>

          <p>
            <strong>Location:</strong>
            {{ lesson.location }}
          </p>

          <p>
            <strong>Price:</strong>
            Rs {{ lesson.price }}
          </p>

          <p>
            <strong>Available Spaces:</strong>
            {{ lesson.space }}
          </p>

          <p
            v-if="lesson.space === 0"
            class="fully-booked"
          >
            Fully Booked
          </p>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.container {
  max-width: 1100px;
  width: 100%;
  margin: 0 auto;
  padding: 30px 20px;
  font-family: Arial, sans-serif;
}

header {
  text-align: center;
  margin-bottom: 40px;
}

h1 {
  font-size: 32px;
  font-weight: bold;
  color: #2563eb;
}

h2 {
  font-size: 24px;
  margin-bottom: 20px;
}

header p {
  font-size: 16px;
  margin-top: 10px;
}

.lesson-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
}

.lesson-card {
  background-color: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 20px;
  color: #1e293b;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.08);
}

.lesson-card h3 {
  font-size: 20px;
  color: #2563eb;
  margin-bottom: 15px;
}

.lesson-card p {
  margin-bottom: 10px;
}

.error {
  color: red;
  text-align: center;
}

.fully-booked {
  color: red;
  font-weight: bold;
}
</style>
