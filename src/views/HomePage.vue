<script setup>
import { ref, watch } from 'vue';
import { 
  IonApp, IonContent, IonSelect, IonSelectOption, IonButton, 
  IonCard, IonCardHeader, IonCardTitle, IonCardContent, 
  IonSpinner, IonText, IonIcon,
} from '@ionic/vue';
import axios from 'axios';
import { star, gitNetwork, alertCircle } from 'ionicons/icons';

const language = ref('');
const repo = ref(null);
const loading = ref(false);
const error = ref(null);

const fetchRepository = async () => {
  if (!language.value) return;

  loading.value = true;
  error.value = null;
  repo.value = null;

  try {
    const response = await axios.get(`https://api.github.com/search/repositories?q=language:${language.value}&sort=stars&order=desc`, {
      headers: {
        Authorization: `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`, 
      }
    });

    console.log(response.data.items);  

    if (response.data.items.length > 0) {
      const randomIndex = Math.floor(Math.random() * response.data.items.length);
      repo.value = response.data.items[randomIndex];
    } else {
      error.value = 'No repositories found';
    }
  } catch (err) {
    error.value = err.message || 'Error fetching repositories';
  } finally {
    loading.value = false;
  }
};

watch(language, fetchRepository);
</script>

<template>
  <ion-app>
    <ion-content class="ion-padding custom-background">
      <ion-card class="custom-card">
        <ion-card-header>
          <ion-card-title class="title">GitHub Repository Finder</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          
          <div class="select-container">
            <ion-select v-model="language" placeholder="Select a Language" class="custom-select">
              <ion-select-option value="javascript">JavaScript</ion-select-option>
              <ion-select-option value="python">Python</ion-select-option>
              <ion-select-option value="java">Java</ion-select-option>
              <ion-select-option value="csharp">C#</ion-select-option>
              <ion-select-option value="ruby">Ruby</ion-select-option>
            </ion-select>
          </div>

          <div class="status-container">
            <ion-text v-if="loading" color="black">Loading, please wait...</ion-text>
            <ion-spinner v-if="loading" color="dark"></ion-spinner>

            <ion-text v-if="error" color="danger">Error fetching repositories</ion-text>

            <ion-button v-if="error" @click="fetchRepository" expand="block" class="retry-button">
              Click to Retry
            </ion-button>
          </div>

          <ion-card v-if="repo" class="result-card">
            <ion-card-header>
              <ion-card-title>
                <a :href="repo.html_url" target="_blank" class="repo-link">{{ repo.name }}</a>
              </ion-card-title>
            </ion-card-header>

            <ion-card-content>
              <p v-if="repo.description">{{ repo.description }}</p>
              <p v-else>No description available</p>
              <div style="display: flex; gap: 10px; align-items: center;">
                <p v-if="repo.language" style="color:chocolate;">{{ repo.language }}</p>
                <ion-icon :icon="star"></ion-icon>
                <p>{{ repo.stargazers_count }}</p>
                <ion-icon :icon="gitNetwork"></ion-icon>
                <p>{{ repo.forks_count }}</p>
                <ion-icon :icon="alertCircle"></ion-icon>
                <p>{{ repo.open_issues_count }}</p>
              </div>
            </ion-card-content>
          </ion-card>

          <ion-button v-if="repo" @click="fetchRepository" expand="block" class="refresh-button">
            Refresh
          </ion-button>

        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-app>
</template>

<style scoped>
.custom-background {
  background-color: #ffffff;
}

.custom-card {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  border-radius: 12px;
  background-color: white;
  box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.05);
  border: 1px solid #f0f0f0;
  border-radius: 2%;
}

.title {
  text-align: center;
  font-size: 22px;
  font-weight: bold;
  color: #333;
}

.select-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.custom-select {
  width: 100%;
  background-color: white;
  border: 1px solid #ddd;
  color: #333;
  border-radius: 4%;
}

.retry-button {
  --background: #ca0d0d;
  --color: #ffffff;
  border: 1px solid #ca0d0d;
  border-radius: 6%;
}

.status-container {
  text-align: center;
  margin-top: 15px;
}

.result-card {
  margin-top: 20px;
  border-left: 5px solid #e0e0e0;
  background-color: white;
}

.repo-link {
  text-decoration: none;
  color: #000000;
  font-weight: bold;
}

.repo-link:hover {
  text-decoration: underline;
}

.refresh-button {
  margin-top: 10px;
  --background: #080808;
  --color: #ffffff;
  border: 1px solid #080808;
}
</style>
