<script>
  import { onMount } from 'svelte';
  import 'bootstrap/dist/css/bootstrap.min.css';

  const apiKey = '16a3974f03c4fcccc82c44efdd5a6a3f';
  let city = '';
  let weatherData = null;

  const fetchWeatherData = async () => {
    try {
      const response = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`);
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      const data = await response.json();
      weatherData = data.list;
      showFilteredData();
    } catch (error) {
      console.error('Error fetching weather data:', error);
    }
  };

  // Function to filter weather data for specific days
  const filterWeatherForSpecificDays = (data, days) => {
    const dayNames = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
    const filteredData = {};
    days.forEach(day => {
      filteredData[dayNames[day]] = data.filter(item => new Date(item.dt_txt).getDay() === day).map(item => {
        return {
          temperature: item.main.temp,
          description: item.weather[0].description,
          humidity: item.main.humidity,
          windSpeed: item.wind.speed
        };
      })[0];
    });
    return filteredData;
  };

  const showFilteredData = () => {
    const filteredData = filterWeatherForSpecificDays(weatherData, [3, 4, 5, 6, 0]);
    weatherData = filteredData;
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
  table {
    width: 100%;
    border-collapse: collapse;
  }
  th, td {
    padding: 10px;
    border: 1px solid #ccc;
  }
</style>

<main class="container mt-4">
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <input type="text" class="form-control mb-2" bind:value={city} placeholder="Enter city name" />
      <button class="btn btn-primary btn-block" on:click={fetchWeatherData}>Get Weather</button>
      
      {#if weatherData}
        <table>
          <thead>
            <tr>
              <th style="color: blue;">Property</th>
              {#each Object.keys(weatherData) as day}
                <th style="color: blue;">{day}</th>
              {/each}
            </tr>
          </thead>
          <tbody>
            {#each Object.keys(weatherData[Object.keys(weatherData)[0]]) as prop}
              <tr>
                <td >{prop}</td>
                {#each Object.values(weatherData) as dayData}
                  <td>{dayData[prop] ? dayData[prop] : '-'}</td>
                {/each}
              </tr>
            {/each}
          </tbody>
        </table>
      {:else}
        <p>No weather data available</p>
      {/if}
    </div>
  </div>
</main>
