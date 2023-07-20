

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
  let savedCities = [];
  let savedCitiesModalStyle = '';
  let savedCitiesModalContent = '';


const saveCity = () => {
  if (cityName && !savedCities.includes(cityName)) {
    savedCities = [...savedCities, cityName];
    showSaveSuccessAlert(); 
  }
};


const showSavedCities = () => {
  const cityList = savedCities.join(', ');
  displaySavedCitiesModal(`Saved Cities: ${cityList}`);
};


const closeSavedCitiesModal = () => {
  savedCitiesModalStyle = '';
};


const displaySavedCitiesModal = (message) => {
  savedCitiesModalStyle = 'display: block;';
  savedCitiesModalContent = message;
};
const removeCity = (cityToRemove) => {
    savedCities = savedCities.filter(city => city !== cityToRemove);
  };


const showSaveSuccessAlert = () => {
  alert('City saved successfully!');
};
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
      currentTemperature = data.list[0].main.temp; 
      cityName = name; 
      showFilteredData();
      closeModal(); 
    } catch (error) {
      console.error('Error fetching weather data:', error);
      displayModal('Please enter a valid city name and could  please check the spelling.');
    }
  };

  
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
      
 
      <button class="btn btn-success btn-block mt-2" on:click={saveCity}>Save City</button>

      
      <button class="btn btn-info btn-block mt-2" on:click={showSavedCities}>Show Saved Cities</button>
      
      {#if locationInfo}
      <h2 style="color: chartreuse;">{locationInfo}</h2>
      {#if weatherData}
        <div class="weather-cards">
          {#each Object.entries(weatherData) as [day, data]}
            <div class="weather-card">
              <h3>{day}</h3>
              <p>Temperature: {data.temperature}°C</p>
              <p>Humidity: {data.humidity}%</p>
              <p>Wind Speed: {data.windSpeed} m/s</p>
              <p>Description: {data.description}</p>
            </div>
          {/each}
        </div>
      {:else}
        <p>No weather data available</p>
      {/if}
    {:else}
      <p>Enter a city name to get weather data.</p>
    {/if}
  </div>
</div>
</main>

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

<div class="modal" tabindex="-1" role="dialog" style="{savedCitiesModalStyle}">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Saved Cities</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close" on:click={closeSavedCitiesModal}>
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        {#if savedCities.length > 0}
          {#each savedCities as city}
            <div>
              <span>{city}</span>
              <button class="btn btn-sm btn-danger" on:click={() => removeCity(city)}>Remove</button>
            </div>
          {/each}
        {:else}
          <p>No saved cities.</p>
        {/if}
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal" on:click={closeSavedCitiesModal}>Close</button>
      </div>
    </div>
  </div>
</div>

<style>
  main.container {
    background-image: url('https://images.unsplash.com/photo-1500382017468-9049fed747ef?fit=crop&w=1300&q=80');
    margin-bottom: -10px;
   margin-left: -10px;
    height: 700px;
   
    width: 1100px;
   
    height: 700px;
    overflow: auto;
    position: relative; 
  }

 .current-weather{
    margin-top: 40px;
    font-size: 25px;
    margin-left: 80px;
  }



  .weather-cards {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;


}

.weather-card {
  border: 1px solid #ef1313;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 20px;
  width: calc(10% - 10px);
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 32%;
  height: 260px;
  background-color: rgb(240, 221, 232);
}

.weather-card h3 {
  font-size: 20px;
  color: purple;
  margin-bottom: 10px;
}

.weather-card p {
  font-size: 16px;
  color: blue;
}
button {
    margin-top: 10px;
  }
  .modal-body div {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

</style>
