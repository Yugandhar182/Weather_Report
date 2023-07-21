
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
   let currenthumidity =null;
  let savedCities = [];
 

  const saveCity = () => {
    if (savedCities.length < 5) {
      if (cityName && !savedCities.some((cityObj) => cityObj.city === cityName)) {
        savedCities = [...savedCities, { city: cityName, temperature: currentTemperature, humidity:currenthumidity }];
        localStorage.setItem('savedCities', JSON.stringify(savedCities));
        showSaveSuccessAlert();
      } else {
        alert('This city is already saved.');
      }
    } else {
      alert('You can only add up to 5 cities.');
    }
  };

const showSaveSuccessAlert = () => {
  alert('City saved successfully!');
};
const fetchWeatherForLastCity = async () => {
    const lastCity = localStorage.getItem('lastCity');
    if (lastCity) {
      city = lastCity;
      await fetchWeatherData();
    }
  };

 
  onMount(async () => {
    const savedCitiesFromLocalStorage = localStorage.getItem('savedCities');
    if (savedCitiesFromLocalStorage) {
      savedCities = JSON.parse(savedCitiesFromLocalStorage);
    }

    await fetchWeatherForLastCity();
  });

  const removeCity = (cityToRemove) => {
  savedCities = savedCities.filter((city) => city.city !== cityToRemove);
  localStorage.setItem('savedCities', JSON.stringify(savedCities));
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
      currenthumidity = data.list[0].main.humidity;

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

      if (daysDiff >= 0 && daysDiff <=5) {
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

  const fetchSavedCitiesWeather = async () => {
  const promises = savedCities.map(async (cityObj) => {
    try {
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/forecast?q=${cityObj.city}&appid=${apiKey}&units=metric`
      );
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      const data = await response.json();
      const currentTemperature = data.list[0].main.temp;
      const humidity = data.list[0].main.humidity;
      const windSpeed = data.list[0].wind.speed;
      return { city: cityObj.city, temperature: currentTemperature, humidity, windSpeed };
    } catch (error) {
      console.error('Error fetching weather data:', error);
      return { city: cityObj.city, temperature: 'N/A', humidity: 'N/A', windSpeed: 'N/A' };
    }
  });

  const results = await Promise.all(promises);
  savedCities = results;
};

 
  onMount(async () => {
    const savedCitiesFromLocalStorage = localStorage.getItem('savedCities');
    if (savedCitiesFromLocalStorage) {
      savedCities = JSON.parse(savedCitiesFromLocalStorage);
    }

    await fetchWeatherForLastCity();
    await fetchSavedCitiesWeather(); 
  });

  const fetchWeatherDataForCity = async (city) => {
    try {
      closeModal(); // Close the modal (if any) before fetching data for the new city
      city = city.trim(); // Trim the city name to remove any leading/trailing spaces
      if (city) {
        // Fetch weather data for the clicked city
        const response = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const data = await response.json();
        weatherData = data.list;
        const { name, state, country } = data.city;
        locationInfo = state ? `Weather in ${name}, ${state}, ${country}` : `Weather in ${name}, ${country}`;
        currentTemperature = data.list[0].main.temp; 
        currenthumidity = data.list[0].main.hum; 
        cityName = name; 
        showFilteredData();
      } else {
        displayModal('Please enter a valid city name.');
      }
    } catch (error) {
      console.error('Error fetching weather data:', error);
      displayModal('Error fetching weather data for the selected city. Please try again later.');
    }
  };
</script>

<div class="weather-heading">
<h1 style="color:Violet;">Weather Report</h1>
</div>

{#if currentTemperature && cityName}
  <div class="current-weather">
    <h3 style="color:green;">Current Temperature in {cityName}: {currentTemperature}°C</h3>
  </div>
{/if}
<main class="container mt-4">
  <div class="row">
    <div class="col-md-6">

      <div class="left-container">
        {#if savedCities.length > 0}
          <h3 style="color: aliceblue;">Added Cities:</h3>
          <ul>
            {#each savedCities as cityObj}
              <li class="card">
                <div class="saved-cities-card-body">
                 
                  <h5  style="color:blue;" class="city-name" on:click={() => fetchWeatherDataForCity(cityObj.city)}>{cityObj.city}</h5>
                  <div class="weather-info">
                    <p>T: {cityObj.temperature}°C , H: {cityObj.humidity}% 
                      <span class="close-box" on:click={() => removeCity(cityObj.city)}>
                        <span class="close-symbol">&times;</span>
                      </span>
                  </div>
                  
                </div>
              </li>
            {/each}
          </ul>
        {:else}
       

          <p style="color: white;">No Added cities.</p>
    
        {/if}
      </div>
    </div>
    <div class="col-md-6">
      <input type="text" class="form-control mb-2" bind:value={city} placeholder="Enter city name" />
      <button class="btn btn-primary btn-block" on:click={fetchWeatherData}>Get Weather</button>
      <span class="spacer"></span>
      <button class="btn btn-success btn-block mt-2" on:click={saveCity}>Add City</button>
      
      {#if locationInfo}
     {#if weatherData}
        <div class="weather-cards">
          {#each Object.entries(weatherData) as [day, data]}
            <div class="weather-card">
              <h3 style="color: crimson;">{day}</h3>
              <p style="color: aliceblue;">Temperature: {data.temperature}°C</p>
              <p style="color: aliceblue;">Humidity: {data.humidity}%</p>
              <p style="color: aliceblue;">Wind Speed: {data.windSpeed} m/s</p>
              <p style="color: aliceblue;" >Description: {data.description}</p>
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

<style>
  main.container {
    background-image: url('https://images.unsplash.com/photo-1580193769210-b8d1c049a7d9?w=1300&q=80');
    margin-bottom: -20px;
   margin-right: 30px;
    height: 600px;
   
    width: 1300px;
   
    height: 700px;
    overflow: auto;
    position: relative; 
  }
  .left-container {
   
    margin-bottom: -10px;
    padding: 20px;
    text-align: center;
    font-size: 24px;
    color: #333;
    height: 700px;
    width: 220px;
  }

 .current-weather{
    margin-top: 40px;
    font-size: 25px;
    margin-left: 600px;
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
  margin-top:20px;
 

}
.weather-card h3 {
  font-size: 20px;
  color: purple;
  
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
  .saved-cities-card-body {
    font-size: 18px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 100%;
  }

  .saved-cities-card-body h5 {
    font-size: 15px;
    font-weight: bold;
    margin-bottom: 5px;
  }

  .weather-info {
    display: flex;
    flex-direction: column;
    margin-bottom: 38px;
    font-size: 15px;
  }

  .card {
    height: 100px;
    width: 180px;
    padding: 15px;
    margin-bottom: 10px;
    margin-left:-60px ;
   
    border: 1px solid #ccc;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  .close-box {
    position: absolute;
    top: 5px;
    right: 5px;
    width: 11px;
    height: 18px;
    border: 1px solid red;
    border-radius: 4px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
  }

  .close-symbol {
    font-size: 16px;
    line-height: 1;
    color: red;
  }
  .city-name {
    cursor: pointer;
    color: blue;
    
  }
  .form-control{
    background-color: #c6daf0;
    margin-top: 5px;
   
  }
  .weather-heading{
    margin-left: 700px;
  }

</style>