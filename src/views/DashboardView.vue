<template>
  <v-container>
    <v-snackbar v-model="snackbarVisible" :timeout="1500" location="top" color="success">
      Your favorite book has been updated!
      <v-btn rounded="xl" variant="outlined" size="small" color="white" class="ml-2" text
        @click="snackbarVisible = false">
        Close
      </v-btn>
    </v-snackbar>
    <v-row class="mb-4">
      <v-col>
        <h1>Welcome, {{ username }}!</h1>
        <p>Discover books and find your all time favorite!</p>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-card-title>
          All Books
        </v-card-title>
        <v-row>
          <v-col cols="4" v-for="book in allBooks" :key="book.id">
            <v-card class="pb-2">
              <v-img :src="book.cover_url" cover></v-img>
              <!-- Add heart icon to the top right corner -->
              <div class="favorites">
                <v-btn v-if="book.isFavorite" size="x-small" icon @click="updateFavorite(book.id)">
                  <v-icon color="red" size="x-large">mdi-heart</v-icon>
                </v-btn>
                <v-btn v-else size="x-small" icon @click="updateFavorite(book.id)">
                  <v-icon size="x-large">mdi-heart-outline</v-icon>
                </v-btn>
              </div>
              <v-card-title> {{ book.title }} </v-card-title>
              <v-card-subtitle> {{ book.author }} </v-card-subtitle>
              <v-card-actions>
                <v-btn variant="text" size="small" color="teal-accent-4" @click="openBookDetails(book)">Learn More</v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
    </v-row>

    <v-dialog v-model="bookDialog" max-width="400px">
      <v-card>
        <!-- Add heart icon to the top right corner -->
        <div class="favorites">
          <v-btn v-if="selectedBook.isFavorite" size="x-small" icon @click="updateFavorite(selectedBook.id)">
            <v-icon color="red" size="x-large">mdi-heart</v-icon>
          </v-btn>
          <v-btn v-else size="x-small" icon @click="updateFavorite(selectedBook.id)">
            <v-icon size="x-large">mdi-heart-outline</v-icon>
          </v-btn>
        </div>
        <v-img :src="selectedBook.cover_url" cover></v-img>
        <v-card-title>{{ selectedBook.title }}</v-card-title>
        <v-card-subtitle>{{ selectedBook.author }}</v-card-subtitle>
        <v-card-text>{{ selectedBook.description }}</v-card-text>
        <v-card-actions>
          <v-btn variant="outlined" @click="bookDialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>


<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const username = localStorage.getItem('username');
const allBooks = ref([]);
const favoritedBooks = ref([]);
const snackbarVisible = ref(false);
const bookDialog = ref(false);
const selectedBook = ref({});

const openBookDetails = (book) => {
  selectedBook.value = book;
  bookDialog.value = true;
};

const updateFavorite = async (bookId) => {
  try {
    await axios.post(`http://localhost:9000/users/${username}/favorites`, { book: bookId });
    for (let book of allBooks.value) {
      book.isFavorite = book.id === bookId;
    }
    snackbarVisible.value = true;  // Show the snackbar
  } catch (error) {
    console.error('Failed to update favorite book:', error);
  }
};



// Fetch all books and user's favorite books
const fetchBooksAndFavorites = async () => {
  try {
    // Fetch all books
    const response = await axios.get('http://localhost:9000/books');
    if (response.data && response.data.data && response.data.data.books) {
      allBooks.value = response.data.data.books;
    }

    // Fetch user's favorite books
    const favResponse = await axios.get(`http://localhost:9000/users/${username}/favorites`);
    if (favResponse.data && favResponse.data.data && favResponse.data.data.favorites) {
      favoritedBooks.value = favResponse.data.data.favorites.book;
    }

    // Mark the favorite books in the allBooks array
    for (let book of allBooks.value) {
      book.isFavorite = favoritedBooks.value.includes(book.id);
    }
  } catch (error) {
    console.error('Failed to fetch data:', error);
  }
};


onMounted(fetchBooksAndFavorites);
</script>


<style>
.favorites {
  position: absolute;
  top: 0.5em;
  right: 0.5em;
  z-index: 100;
}
</style>
