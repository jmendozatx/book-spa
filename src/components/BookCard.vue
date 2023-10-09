<template>
  <v-card class="pb-2">
    <v-img :src="book.cover_url" cover></v-img>
    <div class="favorites">
      <v-btn v-if="book.isFavorite" size="x-small" icon @click="toggleFavorite">
        <v-icon color="red" size="x-large">mdi-heart</v-icon>
      </v-btn>
      <v-btn v-else size="x-small" icon @click="toggleFavorite">
        <v-icon size="x-large">mdi-heart-outline</v-icon>
      </v-btn>
    </div>
    <v-card-title> {{ book.title }} </v-card-title>
    <v-card-subtitle> {{ book.author }} </v-card-subtitle>
    <v-card-text v-if="isDialog">{{ book.description }}</v-card-text>
    <v-card-actions>
      <v-btn v-if="isDialog" variant="outlined" @click="closeDialog">Close</v-btn>
      <v-btn v-else variant="text" size="small" color="teal-accent-4" @click="showDetails">Learn More</v-btn>
    </v-card-actions>
  </v-card>
</template>
  
<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  book: {
    type: Object,
    required: true
  },
  isDialog: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['updateFavorite', 'showBookDetails', 'closeDialog']);

const toggleFavorite = () => {
  emit('updateFavorite', props.book.id);
};

const showDetails = () => {
  emit('showBookDetails', props.book);
};

const closeDialog = () => {
  emit('closeDialog');
};
</script>

  
<style scoped>
.favorites {
  position: absolute;
  top: 0.5em;
  right: 0.5em;
  z-index: 100;
}
</style>
