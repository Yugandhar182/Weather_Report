<script>
  import { onMount } from 'svelte';
  import 'bootstrap/dist/css/bootstrap.min.css';

  const apiKey = '16a3974f03c4fcccc82c44efdd5a6a3f';
  let city = '';
  let locationInfo = '';
  let weatherData = null;

  const fetchWeatherData = async () => {
    try {
      const response = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`);
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      const data = await response.json();
      weatherData = data.list;
      const { name, state, country } = data.city;
      locationInfo = state ? `Weather in ${name}, ${state}, ${country}` : `Weather in ${name}, ${country}`;
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
    const filteredData = filterWeatherForSpecificDays(weatherData, [ 4, 5, 6, 0 , 1]);
    weatherData = filteredData;
  };

  onMount(() => {
    // Fetch weather data on component mount (you can remove this if not needed).
    fetchWeatherData();
  });
</script>


<style>
  main.container {
    
    background-image: url('https://images.pexels.com/photos/2132180/pexels-photo-2132180.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1'); 
    margin-top: 20px;
 
    height: 700px;
    width: 1100px;
  }

  .weather-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  .weather-table th,
  .weather-table td {
    padding: 10px;
    border: 1px solid #ef1313;
  }
  .weather-table th {
    font-size: 18px; 
    color: green; /* You can adjust the color as well */
  }

  .weather-table td {
    font-size: 25px; 
    color: rgb(11, 174, 8); 
  }
</style>

<main class="container mt-4">
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <input type="text" class="form-control mb-2" bind:value={city} placeholder="Enter city name" />
      <button class="btn btn-primary btn-block" on:click={fetchWeatherData}>Get Weather</button>
      
      {#if locationInfo}
        <h2>{locationInfo}</h2>
        {#if weatherData}
          <table class="weather-table">
            <thead>
              <tr>
                <th style="color: green;">Property</th>
                {#each Object.keys(weatherData) as day}
                  <th style="color: blue;">{day}</th>
                {/each}
              </tr>
            </thead>
            <tbody>
              {#each Object.keys(weatherData[Object.keys(weatherData)[0]]) as prop}
                <tr>
                  <td style="color: blue;"  >{prop}</td>
                  {#each Object.values(weatherData) as dayData}
                    <td style="color: crimson;">{dayData[prop] ? dayData[prop] : '-'}</td>
                  {/each}
                </tr>
              {/each}
            </tbody>
          </table>
        {:else}
          <p>No weather data available</p>
        {/if}
      {:else}
        <p>Enter a city name to get weather data.</p>
      {/if}
    </div>
  </div>
</main>
