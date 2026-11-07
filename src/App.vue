
<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'

// ======================================
// LESSON DATA
// ======================================

const lessons = ref([])
const loading = ref(true)
const errorMessage = ref('')
// Store the IDs of lessons added to the cart
const cart = ref([])
// Checkout form
const customerName = ref('')
const customerPhone = ref('')
const checkoutMessage = ref('')

// DISPLAY PREFERENCE: dark mode (stored in this browser)
const isDarkMode = ref(false)

function toggleDarkMode() {
  isDarkMode.value = !isDarkMode.value
  localStorage.setItem('huz-academy-dark-mode', String(isDarkMode.value))
}


// Validate the customer's details using JavaScript
const canCheckout = computed(() => {
  const validName = /^[A-Za-z]+(?: [A-Za-z]+)*$/.test(
    customerName.value.trim()
  )

  const validPhone = /^[0-9]+$/.test(
    customerPhone.value.trim()
  )

  return cart.value.length > 0 && validName && validPhone
})

function prepareCheckout() {
  if (!canCheckout.value) return

  checkoutMessage.value =
    'Details validated. Backend order submission is not yet connected.'
}

// ======================================
// SORTING
// ======================================

const sortType = ref('price')
const sortAscending = ref(true)

const sortedLessons = computed(() => {
  return [...lessons.value].sort((a, b) => {
    let comparison = 0

    if (
      sortType.value === 'price' ||
      sortType.value === 'space'
    ) {
      comparison = a[sortType.value] - b[sortType.value]
    } else {
      comparison = a[sortType.value].localeCompare(
        b[sortType.value]
      )
    }

    return sortAscending.value
      ? comparison
      : -comparison
  })
})

// ======================================
// SHOPPING CART
// ======================================

// Convert saved lesson IDs into lesson objects for display.
// The same lesson may appear more than once when multiple spaces are added.
const cartLessons = computed(() => {
  return cart.value
    .map(id => lessons.value.find(lesson => lesson._id === id))
    .filter(Boolean)
})

// Sum the prices of all selected spaces
const cartTotal = computed(() => {
  return cartLessons.value.reduce(
    (total, lesson) => total + Number(lesson.price),
    0
  )
})

// ======================================
// NAVIGATION
// ======================================

const currentPage = ref('home')
const selectedSubject = ref('')

async function showHome(section = 'top') {
  currentPage.value = 'home'

  await nextTick()

  if (section === 'top') {
    window.scrollTo({
      top: 0,
      behavior: 'smooth'
    })
  } else {
    document.getElementById(section)?.scrollIntoView({
      behavior: 'smooth'
    })
  }
}

function showFaculty() {
  currentPage.value = 'faculty'

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

// Open or close the cart using the same navigation button
function toggleCart() {
  if (cart.value.length === 0) return

  if (currentPage.value === 'cart') {
    showHome('courses')
  } else {
    currentPage.value = 'cart'
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }
}

// Generate Faculty subject tabs from MongoDB
const subjects = computed(() => {
  return [...new Set(
    lessons.value.map(lesson => lesson.topic)
  )].sort((a, b) => a.localeCompare(b))
})

// ======================================
// FETCH LESSONS FROM BACKEND
// ======================================

async function fetchLessons() {
  try {
    const response = await fetch(
      'http://localhost:3000/lessons'
    )

    if (!response.ok) {
      throw new Error('Failed to retrieve lessons')
    }

    lessons.value = await response.json()

    console.log('Lessons retrieved:', lessons.value)
  } catch (error) {
    console.error('Error fetching lessons:', error)

    errorMessage.value =
      'Unable to load lessons. Please try again.'
  } finally {
    loading.value = false
  }
}

// Add one available space to the shopping cart
function addToCart(lesson) {
  // Prevent adding a fully booked lesson
  if (lesson.space <= 0) {
    return
  }

  // Store one entry per selected space
  cart.value.push(lesson._id)

  // Update displayed availability reactively
  lesson.space -= 1

  console.log('Cart:', cart.value)
}

// Remove one selected space and restore the lesson's availability
function removeFromCart(index) {
  const lesson = cartLessons.value[index]
  if (!lesson) return

  cart.value.splice(index, 1)
  lesson.space += 1
}

onMounted(() => {
  // Restore the previously selected theme after refreshing the page
  isDarkMode.value = localStorage.getItem('huz-academy-dark-mode') === 'true'
  fetchLessons()
})
</script>

<template>
  <div class="website" :class="{ 'dark-mode': isDarkMode }">

    <!-- NAVIGATION BAR -->
    <header class="site-header">
      <div class="header-inner">

        <button class="brand" @click="showHome()">
          <span class="brand-symbol">HA</span>

          <span class="brand-text">
            <strong>HUZ ACADEMY</strong>
          </span>
        </button>

        <nav class="nav-tabs" aria-label="Main navigation">
          <button
            :class="{ active: currentPage === 'home' }"
            @click="showHome()"
          >
            Home
          </button>

          <button @click="showHome('courses')">
            Courses
          </button>

          <button
            :class="{ active: currentPage === 'faculty' }"
            @click="showFaculty"
          >
            Faculty
          </button>

          <button @click="showHome('about')">
            About Us
          </button>

          <!-- Disabled until at least one lesson is added -->
          <button
            class="cart-nav-button"
            :class="{ active: currentPage === 'cart' }"
            :disabled="cart.length === 0"
            @click="toggleCart"
          >
            Cart ({{ cart.length }})
          </button>

          <!-- Switch between light and dark mode -->
          <button
            type="button"
            class="theme-toggle"
            :aria-pressed="isDarkMode"
            :aria-label="isDarkMode ? 'Switch to light mode' : 'Switch to dark mode'"
            @click="toggleDarkMode"
          >
            {{ isDarkMode ? '☀ Light mode' : '☾ Dark mode' }}
          </button>
        </nav>

      </div>
    </header>

    <!-- HOMEPAGE -->
    <main v-if="currentPage === 'home'">

      <!-- HERO SECTION -->
      <section class="hero">
        <div class="hero-inner">

          <div class="hero-copy">
            <p class="eyebrow">
              WELCOME TO HUZ ACADEMY
            </p>

            <h1>
              Where knowledge meets opportunity.
            </h1>

            <p class="hero-description">
             This is a fictional academy created by Huzaifa Abawa for the CST3144 Full Stack Development Coursework
            He hopes you enjoy and appreciate the time and effort that went into creating this website!
            </p>

            <div class="hero-actions">
              <button
                class="gold-button"
                @click="showHome('courses')"
              >
                Explore Our Courses
              </button>

              <button
                class="outline-button"
                @click="showFaculty"
              >
                Discover Our Faculty
              </button>
            </div>
          </div>

          <!-- UNIVERSITY HOMEPAGE IMAGE -->
          <div class="hero-image-container">

            <img
              src="/huzuniversity.jpg"
              alt="Huz Academy campus"
              class="hero-image"
            />

            <div class="hero-image-overlay">
              <span>HUZ ACADEMY</span>
              <p>Fictional Academy</p>
            </div>

          </div>

        </div>
      </section>

      <!-- COURSES SECTION -->
      <section id="courses" class="courses-section">
        <div class="section-inner">

          <div class="section-heading">
            <h2>Explore Our Courses</h2>
          </div>

          <!-- SORTING CONTROLS -->
          <div class="sort-controls">

            <label for="sortType">
              Sort by
            </label>

            <select id="sortType" v-model="sortType">
              <option value="topic">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="space">Availability</option>
            </select>

            <label for="sortOrder">
              Order
            </label>

            <select id="sortOrder" v-model="sortAscending">
              <option :value="true">Ascending</option>
              <option :value="false">Descending</option>
            </select>

          </div>
          <p class="cart-summary">
            Items in Cart: {{ cart.length }}
          </p>

          <!-- LOADING AND ERROR HANDLING -->
          <p v-if="loading" class="status-message">
            Loading courses...
          </p>

          <p v-else-if="errorMessage" class="error">
            {{ errorMessage }}
          </p>

          <p
            v-else-if="lessons.length === 0"
            class="status-message"
          >
            No courses are currently available.
          </p>

          <!-- COURSE CARDS -->
          <div v-else class="course-grid">

            <article
              v-for="(lesson, index) in sortedLessons"
              :key="lesson._id"
              class="course-card"
            >

              <!-- COURSE IMAGE FROM EXPRESS SERVER -->
              <img
                :src="`http://localhost:3000/images/${lesson.image}.jpg`"
                :alt="`${lesson.topic} course`"
                class="course-image"
              />

              <div class="card-top">

                <span>
                  COURSE {{ String(index + 1).padStart(2, '0') }}
                </span>

                <span class="course-category">
                  HUZ ACADEMY
                </span>

              </div>

              <div class="card-main">

                <h3>{{ lesson.topic }}</h3>

                <p class="course-location">
                  {{ lesson.location }}, Mauritius
                </p>

              </div>

              <div class="card-bottom">

                <div class="course-details">
                  <span>Fee per place</span>
                  <strong>Rs {{ lesson.price }}</strong>
                </div>

                <div class="course-details">
                  <span>Available places</span>
                  <strong>{{ lesson.space }}</strong>
                </div>

                <p
                  v-if="lesson.space === 0"
                  class="fully-booked"
                >
                  Fully Booked
                </p>

                <!-- ADD TO CART BUTTON -->
                <button
                  class="add-to-cart-button"
                  :disabled="lesson.space <= 0"
                  @click="addToCart(lesson)"
                >
                  {{ lesson.space > 0 ? 'Add to Cart' : 'Fully Booked' }}
                </button>


              </div>

            </article>

          </div>

        </div>
      </section>

      <!-- ABOUT US SECTION -->
      <section id="about" class="about-section">
        <div class="section-inner about-layout">
          <div class="about-heading">
            <p class="eyebrow">ABOUT OUR INSTITUTION</p>
            <h2>Coursework Academy.</h2>
            <div class="gold-divider"></div>
          </div>

          <div class="about-content">
            <p>
              This is a fictional academy created by
              <strong>Huzaifa Abawa</strong> for the
              <strong>CST3144 Full Stack Development Coursework</strong>.
            </p>

            <p>
              He hopes you enjoy and appreciate the time
              and effort that went into creating this website!
            </p>

            <button class="outline-button" @click="showFaculty">
              Explore Our Faculty
            </button>
          </div>
        </div>
      </section>

    </main>

    <!-- FACULTY PAGE -->
    <main v-else-if="currentPage === 'faculty'" class="faculty-page">
      <div class="section-inner">

        <div class="section-heading">

          <p class="section-label">
            OUR TEACHING COMMUNITY
          </p>

          <h1>Faculty Information</h1>
        </div>

        <h2 class="subject-heading">
          Choose a Subject
        </h2>

        <p v-if="loading">
          Loading subjects...
        </p>

        <p v-else-if="errorMessage" class="error">
          {{ errorMessage }}
        </p>

        <!-- SUBJECT TABS -->
        <div v-else class="subject-tabs">

          <button
            v-for="subject in subjects"
            :key="subject"
            :class="{ selected: selectedSubject === subject }"
            @click="selectedSubject = subject"
          >
            {{ subject }}
          </button>

        </div>

        <!-- FACULTY PLACEHOLDER -->
        <div class="faculty-placeholder">

          <p class="section-label">
            FACULTY DIRECTORY
          </p>

          <h3>
            {{ selectedSubject || 'Our Academic Departments' }}
          </h3>

          <p v-if="selectedSubject">
            Teacher profiles for {{ selectedSubject }}
            will appear here once faculty information
            is connected to MongoDB Atlas.
          </p>

          <p v-else>
            Select a subject above to explore
            its teaching department.
          </p>

        </div>

      </div>
    </main>

    <!-- SHOPPING CART PAGE -->
    <main v-else class="cart-page">
      <div class="section-inner">
        <p class="section-label">YOUR SELECTED LESSONS</p>
        <h1>Shopping Cart</h1>

        <p class="section-description">
          Review the spaces you have selected. You can remove a space
          before proceeding to checkout.
        </p>

        <p v-if="cartLessons.length === 0" class="status-message">
          Your shopping cart is empty.
        </p>

        <div v-else class="cart-list">
          <article
            v-for="(lesson, index) in cartLessons"
            :key="`${lesson._id}-${index}`"
            class="cart-item"
          >
            <div class="cart-item-info">
              <h2>{{ lesson.topic }}</h2>
              <p>{{ lesson.location }}, Mauritius</p>
              <span>1 space · Rs {{ lesson.price }}</span>
            </div>

            <button
              class="remove-cart-button"
              @click="removeFromCart(index)"
            >
              Remove
            </button>
          </article>
        </div>

        <div class="cart-total">
          <span>Total ({{ cart.length }} spaces)</span>
          <strong>Rs {{ cartTotal }}</strong>
        </div>

        <button class="back-to-courses-button" @click="showHome('courses')">
          Continue Browsing Lessons
        </button>

        
        <!-- CHECKOUT FORM: validation only; no backend submission yet -->
        <form class="checkout-form" @submit.prevent="prepareCheckout">
          <h2>Checkout</h2>

          <label for="customerName">Full Name</label>
          <input
            id="customerName"
            v-model="customerName"
            type="text"
            placeholder="Enter your full name"
            autocomplete="name"
            @input="checkoutMessage = ''"
          />

          <label for="customerPhone">Phone Number</label>
          <input
            id="customerPhone"
            v-model="customerPhone"
            type="tel"
            inputmode="numeric"
            placeholder="Enter your phone number"
            autocomplete="tel"
            @input="checkoutMessage = ''"
          />

          <button
            type="submit"
            class="checkout-button"
            :disabled="!canCheckout"
          >
            Checkout
          </button>

          <p v-if="checkoutMessage" class="checkout-message" role="status">
            {{ checkoutMessage }}
          </p>
        </form>

      </div>
    </main>

    <!-- FOOTER -->
    <footer class="site-footer">
      <div class="footer-inner">

        <div class="footer-brand">
          <strong>HUZ ACADEMY</strong>
        </div>

        <div class="footer-location">
        </div>

        <small>
          Fictional Academy · CST3144 Coursework
        </small>

      </div>
    </footer>

  </div>
</template>


<style scoped>

/* ======================================
   GENERAL LAYOUT
====================================== */

.website {
  width: 100%;
  min-height: 100vh;
  margin: 0 auto;
  font-family: Arial, Helvetica, sans-serif;
  color: #19382d;
  background: #f5f2e9;
}

button,
select {
  font: inherit;
}

button {
  cursor: pointer;
}

/* Center all main content containers */
.header-inner,
.hero-inner,
.section-inner,
.footer-inner {
  width: 100%;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  padding-left: 28px;
  padding-right: 28px;
}

/* ======================================
   NAVIGATION BAR
====================================== */

.site-header {
  width: 100%;
  background: #102f26;
  border-bottom: 1px solid #456553;
}

.header-inner {
  min-height: 95px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 30px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 14px;
  border: none;
  background: transparent;
  color: #f5f2e9;
  padding: 0;
  text-align: left;
  flex-shrink: 0;
}

.brand-symbol {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #c5ab75;
  border-radius: 50%;
  font: bold 19px Georgia, serif;
  color: #c5ab75;
}

.brand-text strong {
  display: block;
  font-size: 15px;
  letter-spacing: 1.5px;
}

.brand-text small {
  display: block;
  margin-top: 6px;
  font-size: 9px;
  letter-spacing: 2px;
  color: #c5ab75;
}

.nav-tabs {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  flex-wrap: wrap;
  gap: 30px;
}

.nav-tabs button {
  background: transparent;
  border: none;
  padding: 10px 0;
  color: #f5f2e9;
  font-size: 13px;
  transition: color 0.2s ease;
}

.nav-tabs button:hover,
.nav-tabs button.active {
  color: #c5ab75;
}

/* ======================================
   HERO SECTION
====================================== */

.hero {
  width: 100%;
  background: #102f26;
  color: #f5f2e9;
}

.hero-inner {
  padding-top: 85px;
  padding-bottom: 95px;
  display: grid;
  grid-template-columns:
    minmax(0, 1.05fr)
    minmax(0, 0.95fr);
  align-items: center;
  gap: 55px;
}

.hero-copy {
  min-width: 0;
}

.eyebrow,
.section-label {
  color: #c5ab75;
  font-size: 11px;
  font-weight: bold;
  letter-spacing: 2px;
  margin: 0 0 22px;
}

.hero h1 {
  max-width: 620px;
  font: normal clamp(38px, 5vw, 62px) Georgia, serif;
  line-height: 1.15;
  margin: 0 0 25px;
  color: #f5f2e9;
}

.hero-description {
  max-width: 500px;
  color: #cdd7cf;
  line-height: 1.9;
  font-size: 15px;
  margin: 0;
}

.hero-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  margin-top: 35px;
}

/* HERO BUTTONS */

.gold-button,
.outline-button {
  padding: 14px 22px;
  border-radius: 4px;
  font-size: 13px;
  font-weight: bold;
  transition: all 0.2s ease;
}

.gold-button {
  background: #c5ab75;
  border: 1px solid #c5ab75;
  color: #102f26;
}

.gold-button:hover {
  background: #dcc99c;
  border-color: #dcc99c;
}

.outline-button {
  background: transparent;
  border: 1px solid #8aa392;
  color: #f5f2e9;
}

.outline-button:hover {
  background: rgba(255, 255, 255, 0.1);
}

/* ======================================
   UNIVERSITY HERO IMAGE
====================================== */

.hero-image-container {
  position: relative;
  width: 100%;
  height: 430px;
  min-width: 0;
  overflow: hidden;
  border: 1px solid #c5ab75;
  border-radius: 5px;
  box-shadow: 0 18px 45px rgba(0, 0, 0, 0.25);
}

.hero-image {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  filter: brightness(0.85) saturate(0.9);
  transition: transform 0.4s ease;
}

.hero-image-container:hover .hero-image {
  transform: scale(1.04);
}

.hero-image-container::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to top,
    rgba(10, 37, 27, 0.9),
    rgba(10, 37, 27, 0.05) 65%
  );
  pointer-events: none;
}

.hero-image-overlay {
  position: absolute;
  bottom: 25px;
  left: 25px;
  right: 25px;
  z-index: 1;
  color: #f5f2e9;
}

.hero-image-overlay span {
  font-family: Georgia, serif;
  font-size: 23px;
  letter-spacing: 2px;
}

.hero-image-overlay p {
  color: #c5ab75;
  font-size: 12px;
  letter-spacing: 1px;
  margin-top: 8px;
}

/* ======================================
   COURSES SECTION
====================================== */

.courses-section {
  width: 100%;
  background: #f5f2e9;
  scroll-margin-top: 20px;
}

.section-inner {
  padding-top: 85px;
  padding-bottom: 85px;
}

.section-heading {
  max-width: 750px;
  margin-bottom: 35px;
}

.courses-section .section-label,
.faculty-page .section-label {
  color: #907342;
}

.section-heading h2,
.faculty-page h1 {
  font: normal 42px Georgia, serif;
  color: #19382d;
  margin: 0 0 20px;
}

.section-description {
  color: #66756b;
  line-height: 1.9;
  max-width: 650px;
  margin: 0;
  font-size: 15px;
}

/* ======================================
   SORTING CONTROLS
====================================== */

.sort-controls {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 20px;
}

.sort-controls label {
  font-size: 13px;
  font-weight: bold;
  color: #19382d;
}

.sort-controls select {
  background: #ffffff;
  color: #19382d;
  border: 1px solid #ced6cb;
  border-radius: 4px;
  padding: 11px 13px;
  min-width: 130px;
  cursor: pointer;
}

.sort-controls select:focus {
  outline: 2px solid #c5ab75;
}

/* ======================================
   CART SUMMARY - NEW
====================================== */

.cart-summary {
  display: inline-block;
  margin: 0 0 30px;
  padding: 11px 16px;
  background: #e7ede6;
  color: #19382d;
  border: 1px solid #cddbcf;
  border-left: 4px solid #c5ab75;
  border-radius: 4px;
  font-size: 13px;
  font-weight: bold;
}

/* ======================================
   COURSE CARDS
====================================== */

.course-grid {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(245px, 1fr)
  );
  gap: 22px;
}

.course-card {
  background: #ffffff;
  border: 1px solid #e2e5dc;
  border-top: 3px solid #244c3b;
  border-radius: 5px;
  padding: 26px;
  min-width: 0;
  display: flex;
  flex-direction: column;
  transition:
    box-shadow 0.2s ease,
    transform 0.2s ease;
}

.course-card:hover {
  box-shadow: 0 10px 25px rgba(16, 47, 38, 0.1);
  transform: translateY(-3px);
}

/* COURSE IMAGES */

.course-image {
  display: block;
  width: 100%;
  height: 190px;
  object-fit: cover;
  border-radius: 4px;
  margin-bottom: 22px;
  background: #e8eae3;
}

/* COURSE INFORMATION */

.card-top {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px;
  color: #788579;
  font-size: 10px;
  letter-spacing: 1px;
}

.course-category {
  color: #907342;
}

.card-main {
  flex: 1;
  padding-top: 18px;
  padding-bottom: 20px;
}

.course-card h3 {
  font: normal 27px Georgia, serif;
  color: #19382d;
  margin: 12px 0;
}

.course-location {
  color: #65776a;
  font-size: 13px;
  line-height: 1.7;
}

.card-bottom {
  margin-top: auto;
}

.course-details {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 13px 0;
  border-top: 1px solid #e8eae3;
  font-size: 13px;
}

.course-details span {
  color: #66756b;
}

.course-details strong {
  color: #19382d;
}

.fully-booked {
  color: #a52e2e;
  font-weight: bold;
  font-size: 13px;
}

.error {
  color: #a52e2e;
  font-weight: bold;
}

.status-message {
  color: #66756b;
  padding: 25px 0;
}

/* ======================================
   ADD TO CART BUTTON - NEW
====================================== */

.add-to-cart-button {
  display: block;
  width: 100%;
  margin-top: 18px;
  padding: 14px 18px;

  background: #19382d;
  color: #f5f2e9;

  border: 1px solid #19382d;
  border-radius: 4px;

  font-size: 13px;
  font-weight: bold;
  letter-spacing: 0.4px;

  cursor: pointer;
  transition:
    background 0.2s ease,
    transform 0.2s ease;
}

/* Button hover effect */
.add-to-cart-button:hover:not(:disabled) {
  background: #2d5944;
  transform: translateY(-2px);
}

/* Button click effect */
.add-to-cart-button:active:not(:disabled) {
  transform: translateY(0);
}

/* Fully booked / disabled button */
.add-to-cart-button:disabled {
  background: #e5e7e2;
  border-color: #d0d5ce;
  color: #788579;
  cursor: not-allowed;
  transform: none;
}

/* Keyboard accessibility */
.add-to-cart-button:focus-visible {
  outline: 2px solid #c5ab75;
  outline-offset: 3px;
}

/* ======================================
   ABOUT US SECTION
====================================== */

.about-section {
  width: 100%;
  background: #244c3b;
  color: #f5f2e9;
  scroll-margin-top: 20px;
}

.about-layout {
  display: grid;
  grid-template-columns:
    minmax(0, 1fr)
    minmax(0, 1.3fr);
  gap: 70px;
}

.about-layout h2 {
  font: normal 46px Georgia, serif;
  line-height: 1.2;
  color: #f5f2e9;
  margin: 0;
}

.gold-divider {
  width: 65px;
  height: 2px;
  background: #c5ab75;
  margin-top: 30px;
}

.about-content p {
  line-height: 1.9;
  color: #e0e8dd;
  margin: 0 0 22px;
  font-size: 15px;
}

.about-content button {
  margin-top: 12px;
}

/* ======================================
   FACULTY PAGE
====================================== */

.faculty-page {
  width: 100%;
  background: #f5f2e9;
  min-height: 70vh;
}

.faculty-page h1 {
  margin: 0 0 20px;
}

.subject-heading {
  font: normal 27px Georgia, serif;
  color: #19382d;
  margin: 45px 0 25px;
}

.subject-tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 40px;
}

.subject-tabs button {
  border: 1px solid #aebcaf;
  border-radius: 4px;
  background: #ffffff;
  color: #19382d;
  padding: 12px 18px;
  font-size: 13px;
  transition: all 0.2s ease;
}

.subject-tabs button:hover,
.subject-tabs button.selected {
  background: #19382d;
  color: #ffffff;
}

.faculty-placeholder {
  padding: 45px;
  background: #ffffff;
  border-left: 4px solid #c5ab75;
  max-width: 850px;
}

.faculty-placeholder h3 {
  font: normal 30px Georgia, serif;
  color: #19382d;
  margin: 0 0 18px;
}

.faculty-placeholder p:last-child {
  color: #66756b;
  line-height: 1.8;
  margin-bottom: 0;
}

/* ======================================
   SHOPPING CART PAGE
====================================== */

.nav-tabs .cart-nav-button {
  border: 1px solid #c5ab75;
  border-radius: 4px;
  padding: 10px 14px;
}

.nav-tabs .cart-nav-button:disabled {
  border-color: #64796b;
  color: #8a9c90;
  cursor: not-allowed;
}

.cart-page {
  background: #f5f2e9;
  min-height: 70vh;
}

.cart-page h1 {
  font: normal 42px Georgia, serif;
  color: #19382d;
  margin: 0 0 20px;
}

.cart-list {
  display: grid;
  gap: 14px;
  margin-top: 32px;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  padding: 22px;
  border: 1px solid #e2e5dc;
  border-left: 4px solid #c5ab75;
  border-radius: 4px;
  background: white;
}

.cart-item h2 {
  font: normal 23px Georgia, serif;
  color: #19382d;
  margin: 0 0 10px;
}

.cart-item p {
  color: #66756b;
  margin: 0 0 8px;
}

.cart-item span {
  color: #19382d;
  font-weight: bold;
  font-size: 13px;
}

.remove-cart-button,
.back-to-courses-button {
  border-radius: 4px;
  padding: 12px 18px;
  font-size: 13px;
  font-weight: bold;
}

.remove-cart-button {
  border: 1px solid #b74a4a;
  color: #a52e2e;
  background: white;
}

.remove-cart-button:hover {
  background: #fff2f2;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  padding: 22px 0;
  margin-top: 15px;
  border-top: 2px solid #c5ab75;
  font-size: 18px;
  color: #19382d;
}

.back-to-courses-button {
  background: #19382d;
  color: #f5f2e9;
  border: 1px solid #19382d;
}

.back-to-courses-button:hover {
  background: #2d5944;
}

@media (max-width: 480px) {
  .cart-item {
    align-items: stretch;
    flex-direction: column;
  }
}

/* ======================================
   CHECKOUT FORM
====================================== */

.checkout-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
  width: 100%;
  max-width: 520px;
  margin-top: 35px;
  padding: 30px;
  background: #ffffff;
  border: 1px solid #e2e5dc;
  border-top: 4px solid #c5ab75;
  border-radius: 5px;
}

.checkout-form h2 {
  font: normal 28px Georgia, serif;
  color: #19382d;
  margin: 0 0 8px;
}

.checkout-form label {
  font-size: 13px;
  font-weight: bold;
  color: #19382d;
}

.checkout-form input {
  width: 100%;
  padding: 13px;
  border: 1px solid #ced6cb;
  border-radius: 4px;
  background: #ffffff;
  color: #19382d;
  font: inherit;
}

.checkout-form input:focus-visible {
  outline: 2px solid #c5ab75;
  outline-offset: 2px;
}

.checkout-button {
  margin-top: 12px;
  padding: 14px;
  background: #19382d;
  color: #ffffff;
  border: 1px solid #19382d;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
}

.checkout-button:hover:not(:disabled) {
  background: #2d5944;
}

.checkout-button:disabled {
  background: #cbd5ce;
  border-color: #cbd5ce;
  color: #66756b;
  cursor: not-allowed;
}

.checkout-message {
  margin: 5px 0 0;
  color: #19382d;
  font-size: 13px;
  line-height: 1.5;
}

@media (max-width: 480px) {
  .checkout-form {
    padding: 22px;
  }
}

/* ======================================
   FOOTER
====================================== */

.site-footer {
  width: 100%;
  background: #0b241d;
  color: #f5f2e9;
}

.footer-inner {
  padding-top: 35px;
  padding-bottom: 35px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  flex-wrap: wrap;
  font-size: 12px;
}

.footer-brand {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.footer-brand strong {
  font-size: 14px;
  letter-spacing: 1.5px;
}

.footer-brand span,
.footer-location,
.footer-inner small {
  color: #acbbaf;
}

/* ======================================
   LIGHT / DARK MODE TOGGLE
   Only applies when the root .website has .dark-mode
====================================== */

.nav-tabs .theme-toggle {
  padding: 9px 13px;
  border: 1px solid #c5ab75;
  border-radius: 5px;
  color: #f5f2e9;
  white-space: nowrap;
}

.nav-tabs .theme-toggle:hover,
.nav-tabs .theme-toggle:focus-visible {
  background: #c5ab75;
  color: #102f26;
}

.website.dark-mode {
  background: #101e18;
  color: #edf3eb;
}

.dark-mode .site-header,
.dark-mode .hero {
  background: #0a1c15;
}

.dark-mode .about-section {
  background: #17372a;
}

.dark-mode .site-footer {
  background: #07150f;
}

.dark-mode .courses-section,
.dark-mode .faculty-page,
.dark-mode .cart-page {
  background: #101e18;
}

.dark-mode .course-card,
.dark-mode .faculty-placeholder,
.dark-mode .cart-item,
.dark-mode .checkout-form {
  background: #1d3329;
  border-color: #3a5447;
}

.dark-mode .section-heading h2,
.dark-mode .faculty-page h1,
.dark-mode .cart-page h1,
.dark-mode .course-card h3,
.dark-mode .subject-heading,
.dark-mode .faculty-placeholder h3,
.dark-mode .cart-item h2,
.dark-mode .checkout-form h2,
.dark-mode .checkout-form label,
.dark-mode .sort-controls label,
.dark-mode .course-details strong,
.dark-mode .cart-item span,
.dark-mode .cart-total,
.dark-mode .checkout-message {
  color: #f2f6ed;
}

.dark-mode .section-description,
.dark-mode .course-location,
.dark-mode .course-details span,
.dark-mode .card-top,
.dark-mode .status-message,
.dark-mode .cart-item p,
.dark-mode .faculty-placeholder p:last-child {
  color: #bacbbd;
}

.dark-mode .course-category,
.dark-mode .courses-section .section-label,
.dark-mode .faculty-page .section-label {
  color: #ddc28a;
}

.dark-mode .course-details,
.dark-mode .cart-total {
  border-color: #42604f;
}

.dark-mode .cart-summary {
  background: #264535;
  color: #f2f6ed;
  border-color: #476652;
  border-left-color: #c5ab75;
}

.dark-mode .sort-controls select,
.dark-mode .checkout-form input,
.dark-mode .subject-tabs button {
  background: #14271e;
  color: #edf3eb;
  border-color: #52715e;
}

.dark-mode .subject-tabs button:hover,
.dark-mode .subject-tabs button.selected {
  background: #c5ab75;
  color: #102f26;
}

.dark-mode .add-to-cart-button,
.dark-mode .checkout-button,
.dark-mode .back-to-courses-button {
  background: #c5ab75;
  border-color: #c5ab75;
  color: #102f26;
}

.dark-mode .add-to-cart-button:hover:not(:disabled),
.dark-mode .checkout-button:hover:not(:disabled),
.dark-mode .back-to-courses-button:hover {
  background: #dac48f;
  border-color: #dac48f;
}

.dark-mode .add-to-cart-button:disabled,
.dark-mode .checkout-button:disabled {
  background: #344b3e;
  border-color: #435a4c;
  color: #adc0ae;
}

.dark-mode .remove-cart-button {
  background: #302321;
  border-color: #b9756d;
  color: #ffbab0;
}

.dark-mode .remove-cart-button:hover {
  background: #4b2e2b;
}

.dark-mode .fully-booked,
.dark-mode .error {
  color: #ffafa8;
}

/* ======================================
   RESPONSIVE DESIGN
====================================== */

@media (max-width: 900px) {
  .hero-inner {
    gap: 30px;
  }

  .hero-image-container {
    height: 350px;
  }

  .nav-tabs {
    gap: 18px;
  }
}

@media (max-width: 750px) {
  .header-inner {
    flex-direction: column;
    align-items: center;
    padding-top: 20px;
    padding-bottom: 20px;
  }

  .nav-tabs {
    justify-content: center;
    gap: 20px;
  }

  .hero-inner {
    grid-template-columns: 1fr;
    padding-top: 55px;
    padding-bottom: 65px;
  }

  .hero-image-container {
    height: 300px;
  }

  .hero-image-overlay span {
    font-size: 20px;
  }

  .section-inner {
    padding-top: 60px;
    padding-bottom: 60px;
  }

  .about-layout {
    grid-template-columns: 1fr;
    gap: 35px;
  }

  .hero h1 {
    font-size: 43px;
  }

  .about-layout h2 {
    font-size: 38px;
  }

  .footer-inner {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }
}

@media (max-width: 480px) {
  .header-inner,
  .hero-inner,
  .section-inner,
  .footer-inner {
    padding-left: 20px;
    padding-right: 20px;
  }

  .nav-tabs {
    gap: 13px;
  }

  .nav-tabs button {
    font-size: 12px;
  }

  .hero h1 {
    font-size: 36px;
  }

  .hero-image-container {
    height: 260px;
  }

  .course-grid {
    grid-template-columns: 1fr;
  }

  .section-heading h2,
  .faculty-page h1 {
    font-size: 34px;
  }

  .faculty-placeholder {
    padding: 25px;
  }

  .add-to-cart-button {
    padding: 15px;
  }
}

</style>

