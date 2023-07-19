<script>
  import { onMount } from 'svelte';
  import axios from 'axios';
  import 'bootstrap/dist/css/bootstrap.min.css';

  const apiKey = '16a3974f03c4fcccc82c44efdd5a6a3f';
  let city = '';
  let weatherData = null;

  const fetchWeatherData = async () => {
    try {
      const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`);
      weatherData = response.data;
    } catch (error) {
      console.error('Error fetching weather data:', error);
    }
  };

  onMount(() => {
    // Fetch weather data on component mount (you can remove this if not needed).
    fetchWeatherData();
  });
</script>

<style>
 
  

  
  main.container {
    margin-top: 20px;
  }
</style>

<main class="container mt-4">
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <input type="text" class="form-control mb-2" bind:value={city} placeholder="Enter city name" />
      <button class="btn btn-primary btn-block" on:click={fetchWeatherData}>Get Weather</button>

      {#if weatherData}
        <div class="mt-4">
          <h2>Weather for {weatherData.name}</h2>
          <p>Temperature: {weatherData.main.temp}°C</p>
          <p>Description: {weatherData.weather[0].description}</p>
          <!-- Add more weather data as needed -->
        </div>
      {:else}
        <p>No weather data available</p>
      {/if}
    </div>
  </div>
</main>
