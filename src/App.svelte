

<script>
  import { onMount } from 'svelte';
  import 'bootstrap/dist/css/bootstrap.min.css';

  const apiKey = '16a3974f03c4fcccc82c44efdd5a6a3f';
  let city = '';
  let locationInfo = '';
  let weatherData = null;
  let modalStyle = '';
  let errorMessage = '';
  let currentTemperature = null;
  let cityName = null;

  const displayModal = (message) => {
    errorMessage = message;
    modalStyle = 'display: block;';
  };

  const closeModal = () => {
    modalStyle = '';
    errorMessage = '';
  };

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
      currentTemperature = data.list[0].main.temp; // Set current temperature
      cityName = name; // Set city name
      showFilteredData();
      closeModal(); // Hide the modal if the fetch is successful
    } catch (error) {
      console.error('Error fetching weather data:', error);
      displayModal('Please enter a valid city name and could  please check the spelling.');
    }
  };

  // Function to filter weather data for the next five days
  const filterWeatherForNextFiveDays = (data) => {
    const filteredData = {};
    const currentDate = new Date();
    currentDate.setHours(0, 0, 0, 0); 

    data.forEach((item) => {
      const itemDate = new Date(item.dt_txt);
      itemDate.setHours(0, 0, 0, 0);

      const timeDiff = itemDate.getTime() - currentDate.getTime();
      const daysDiff = timeDiff / (1000 * 3600 * 24);

      if (daysDiff >= 0 && daysDiff <= 4) {
        const dayName = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'][itemDate.getDay()];
        if (!filteredData[dayName]) {
          filteredData[dayName] = {
            temperature: item.main.temp,
            description: item.weather[0].description,
            humidity: item.main.humidity,
            windSpeed: item.wind.speed,
          };
        }
      }
    });

    return filteredData;
  };

  const showFilteredData = () => {
    const filteredData = filterWeatherForNextFiveDays(weatherData);
    weatherData = filteredData;
  };

  onMount(() => {
    closeModal(); 
  });
</script>


{#if currentTemperature && cityName}
  <div class="current-weather">
    <h3 style="color:green;">Current Temperature in {cityName}: {currentTemperature}°C</h3>
  </div>
{/if}

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
                <th style="color: purple;">Property</th>
                {#each Object.keys(weatherData) as day}
                  <th style="color: purple;">{day}</th>
                {/each}
              </tr>
            </thead>
            <tbody>
              {#each Object.keys(weatherData[Object.keys(weatherData)[0]]) as prop}
                <tr>
                  <td style="color: blue;">{prop}</td>
                  {#each Object.values(weatherData) as dayData}
                    <td>
                      {#if dayData[prop]}
                        {#if prop === 'description'}
                          <i class="weather-icon fa fa-{dayData[prop].toLowerCase()}"></i>
                          <span>{dayData[prop]}</span>
                        {:else}
                          {dayData[prop]}
                        {/if}
                      {:else}
                        -
                      {/if}
                    </td>
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

  <div class="modal" tabindex="-1" role="dialog" style="{modalStyle}">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Error</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close" on:click={closeModal}>
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          {errorMessage}
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal" on:click={closeModal}>Close</button>
        </div>
      </div>
    </div>
  </div>
</main>


<style>
  main.container { 
    background-image: url('https://images.unsplash.com/photo-1500382017468-9049fed747ef?fit=crop&w=1300&q=80'); 
    margin-bottom: -30px;
    height: 700px;
    width: 1300px;
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
    color: green; 
  }

  .weather-table td {
    font-size: 25px; 
    color: rgb(11, 174, 8); 
  }

  .weather-icon {
    font-size: 25px;
  }
  .current-weather{
    margin-top: 40px;
    font-size: 25px;
    margin-left: 80px;
  }
</style>
