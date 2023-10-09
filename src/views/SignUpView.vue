<template>
  <v-container fluid class="signup-container fill-height">
    <v-row justify="center">
      <v-col cols="12" md="6">
        <v-card class="pa-4">
          <v-card-title class="text-center">
            <img src="@/assets/logo.png" alt="Book Logo" />

            <h2>Join the Community!</h2>
          </v-card-title>

          <v-alert type="error" v-if="errorMessage" class="mb-4" @click="errorMessage = ''">
            {{ errorMessage }}
          </v-alert>

          <v-form ref="form" @submit.prevent="formValidation">
            <!-- Username Field -->
            <v-text-field variant="outlined" label="Username" v-model="username" required :counter="20"
              :rules="usernameRules" placeholder="Choose a username"
              hint="This will be your unique identity on BeyondTheBook."></v-text-field>

            <!-- Password Field -->
            <v-text-field variant="outlined" label="Password" v-model="password" type="password" required
              :rules="passwordRules" placeholder="Choose a strong password"></v-text-field>

            <!-- Favorite Book Selection -->
            <v-select variant="outlined" :items="bookTitles" item-title="text" item-value="value"
              :rules="favoriteBookRules" label="Which of these is your favorite book?" v-model="favoriteBookId"
              hint="Don't worry, you can always update this later in your profile."
              placeholder="Select your favorite book">
            </v-select>

            <!-- Signup Button -->
            <div class="btn-container">
              <v-btn size="large" rounded="xl" color="primary" type="submit">Join Now</v-btn>
            </div>
          </v-form>

          <!-- Footer/Login Navigation -->
          <div class="text-center mt-4">
            Already a member? <router-link to="/login">Log in here</router-link>.
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';


// Data properties
const username = ref('');
const password = ref('');
const books = ref([]);
const errorMessage = ref("");
const form = ref(null);

// Fetch books on component mount
onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:9000/books');
    if (response.data && response.data.data && response.data.data.books) {
      books.value = response.data.data.books;
    }
  } catch (error) {
    console.error('Failed to fetch books:', error);
  }
});

const favoriteBookId = ref(null); // This should be set to the ID of the selected book

// Computed property to transform books into a format that v-select expects
const bookTitles = computed(() => {
  return books.value.map(book => ({
    text: book.title,
    value: book.id
  }));
});

const router = useRouter();

// Form submission method
const formValidation = () => {
  errorMessage.value = "";

  if (username.value.length === 0 || password.value.length === 0 || favoriteBookId.value.length === 0) {
    errorMessage.value = "Please fill out all fields";
  } else {
    submitForm();
  }
}
const submitForm = async () => {
  try {
    // Create the new user
    const newUserResponse = await axios.post('http://localhost:9000/users/new', {
      username: username.value,
      password: password.value
    });

    if (newUserResponse.status !== 201) {
      throw new Error("Error creating new user");
    }

    // Log the user in
    const loginResponse = await axios.post('http://localhost:9000/login', {
      username: username.value,
      password: password.value
    });

    if (loginResponse.status !== 200) {
      throw new Error("Login failed");
    }

    // Save the user's favorite book
    const favoriteResponse = await axios.post(`http://localhost:9000/users/${username.value}/favorites`, {
      book: favoriteBookId.value
    });

    if (favoriteResponse.status !== 200) {
      throw new Error("Failed to save favorite book");
    }

    // Store the username in localStorage
    localStorage.setItem('username', username.value);

    // Navigate to dashboard or show a success message
    router.push('/dashboard')

  } catch (error) {
    console.error("Error during signup:", error);
    errorMessage.value = error;
  }
};

// Password validation rules
const usernameRules = [
  v => !!v || 'Username is required',
  v => (v && v.length <= 20) || 'Username must be less than 20 characters'
];

const passwordRules = [
  v => !!v || 'Password is required',
  v => (v && v.length >= 8) || 'Password must be at least 8 characters'
];

const favoriteBookRules = [
  v => !!v || 'You must select a favorite book.'
];
</script>

<style scoped>
.btn-container {
  text-align: center;
}
</style>
