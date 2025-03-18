<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Weather App</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-item>
        <ion-label position="floating" style="margin-bottom: 15px;">Ubicación</ion-label>
        <ion-input v-model="location" placeholder="Ingresa una ubicación"></ion-input>
      </ion-item>
      <ion-button expand="full" @click="getWeatherByLocation">Obtener Clima</ion-button>
      <ion-button expand="full" @click="getCurrentLocation">Usar Ubicación Actual</ion-button>

      <ion-card v-if="weatherData">
        <ion-card-header>
          <ion-card-title>{{ weatherData.resolvedAddress }}</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <p>Temperatura: {{ weatherData.currentConditions.temp }}°C</p>
          <p>Velocidad del Viento: {{ weatherData.currentConditions.windspeed }} km/h</p>
          <p>Probabilidad de Lluvia: {{ weatherData.currentConditions.precipprob }}%</p>
          <p>Condiciones: {{ weatherData.currentConditions.conditions }}</p>
        </ion-card-content>
      </ion-card>

       <!-- Sección para las últimas 24 horas -->
       <ion-card v-if="past24Hours.length > 0">
        <ion-card-header>
          <ion-card-title>Últimas 24 Horas</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <ion-list>
            <ion-item v-for="hour in past24Hours" :key="hour.datetimeEpoch">
              <ion-label>
                <h2>{{ new Date(hour.datetimeEpoch * 1000).toLocaleTimeString() }}</h2>
                <p>Temperatura: {{ hour.temp }}°C</p>
                <p>Viento: {{ hour.windspeed }} km/h</p>
                <p>Lluvia: {{ hour.precipprob }}%</p>
                <p>Condiciones: {{ hour.conditions }}</p>
              </ion-label>
            </ion-item>
          </ion-list>
        </ion-card-content>
      </ion-card>

      <!-- Sección para las próximas 24 horas -->
      <ion-card v-if="next24Hours.length > 0">
        <ion-card-header>
          <ion-card-title>Próximas 24 Horas</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <ion-list>
            <ion-item v-for="hour in next24Hours" :key="hour.datetimeEpoch">
              <ion-label>
                <h2>{{ new Date(hour.datetimeEpoch * 1000).toLocaleTimeString() }}</h2>
                <p>Temperatura: {{ hour.temp }}°C</p>
                <p>Viento: {{ hour.windspeed }} km/h</p>
                <p>Lluvia: {{ hour.precipprob }}%</p>
                <p>Condiciones: {{ hour.conditions }}</p>
              </ion-label>
            </ion-item>
          </ion-list>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonLabel, IonInput, IonButton, IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonList } from '@ionic/vue';
import { Geolocation } from '@capacitor/geolocation';

// Interfaces para los datos de la API
interface WeatherData {
  resolvedAddress: string;
  currentConditions: {
    temp: number;
    windspeed: number;
    precipprob: number;
    conditions: string;
  };
}

interface WeatherData24Hours {
  resolvedAddress: string;
  currentConditions: {
    temp: number;
    windspeed: number;
    precipprob: number;
    conditions: string;
  };
  days: {
    datetimeEpoch: number;
    hours: {
      datetimeEpoch: number;
      temp: number;
      windspeed: number;
      precipprob: number;
      conditions: string;
    }[];
  }[];
}

// Variables reactivas
const location = ref('');
const weatherData = ref<WeatherData | null>(null);
const weatherData24Hours = ref<WeatherData24Hours | null>(null);

// Métodos
const getWeatherByLocation = async () => {
  if (location.value) {
    await fetchWeatherData(location.value);
    await fetchWeatherData2(location.value);
  }
};

const getCurrentLocation = async () => {
  try {
    const coordinates = await Geolocation.getCurrentPosition();
    const currentLocation = `${coordinates.coords.latitude},${coordinates.coords.longitude}`;
    await fetchWeatherData(currentLocation);
    await fetchWeatherData2(currentLocation);
  } catch (error) {
    console.error('Error obteniendo la ubicación:', error);
  }
};

const fetchWeatherData = async (location: string) => {
  const apiKey = 'N46PKQYFY3ZPUYY8TFGAEG8PU'; // Reemplaza con tu API Key
  const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${location}?unitGroup=metric&include=current&key=${apiKey}`;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Error al obtener los datos del clima');
    }
    const data = await response.json();
    weatherData.value = data;
  } catch (error) {
    console.error('Error fetching weather data:', error);
  }
};

const fetchWeatherData2 = async (location: string) => {
  const apiKey = 'N46PKQYFY3ZPUYY8TFGAEG8PU'; // Reemplaza con tu API Key
  const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${location}?unitGroup=metric&include=hours&key=${apiKey}`;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Error al obtener los datos del clima');
    }
    const data = await response.json();
    weatherData24Hours.value = data;
  } catch (error) {
    console.error('Error fetching weather data:', error);
  }
};

const past24Hours = computed(() => {
  if (!weatherData24Hours.value || !weatherData24Hours.value.days) return [];
  const now = Math.floor(Date.now() / 1000); // Tiempo actual en segundos
  const hours = weatherData24Hours.value.days.flatMap(day => day.hours || []);
  return hours.filter(hour => hour.datetimeEpoch && hour.datetimeEpoch >= now - 86400 && hour.datetimeEpoch < now);
});

const next24Hours = computed(() => {
  if (!weatherData24Hours.value || !weatherData24Hours.value.days) return [];
  const now = Math.floor(Date.now() / 1000); // Tiempo actual en segundos
  const hours = weatherData24Hours.value.days.flatMap(day => day.hours || []);
  return hours.filter(hour => hour.datetimeEpoch && hour.datetimeEpoch >= now && hour.datetimeEpoch < now + 86400);
});

</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>