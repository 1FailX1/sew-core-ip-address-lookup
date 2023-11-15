<script setup>
import { ref } from 'vue'

const dataReceived = ref(false);
const errormessage = ref('')

let generalData = ref("");
let sunriseData = ref("");
let timeData = ref("");
let weatherData = ref("");

function lookupInput() {
    const input = userInput.value;
    document.getElementById("userInput").value = "";

    //General request
    var endpoint = 'http://ip-api.com/json/' + input;

    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            var response = JSON.parse(this.responseText);
            if (response.status !== 'success') {
                dataReceived.value = false;
                errormessage.value = 'Invalid input';
                console.log('query failed: ' + response.message);
                return
            }
            errormessage.value = '';
            generalData.value = response;
            lookupSunriseAndFall();
        } else {
            dataReceived.value = false;
        }
    };
    xhr.open('GET', endpoint, true);
    xhr.send();
}

function lookupSunriseAndFall() {
    //Sunrise / Sunset request
    var endpoint2 = 'https://api.sunrise-sunset.org/json?' + "lat=" + generalData.value.lat + "&lng=" + generalData.value.lon;
    console.log(endpoint2)
    var xhr2 = new XMLHttpRequest();
    xhr2.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            var response = JSON.parse(this.responseText);
            if (response.status !== 'OK') {
                console.log('Sunrise/-fall query failed!');
                return
            }
            console.log(response)
            sunriseData.value = response;
            lookupLocalTime();
        }
    };
    xhr2.open('GET', endpoint2, true);
    xhr2.send();
}

function lookupLocalTime() {
    //API Key: XH0FAMNCPCSD

    var endpoint3 = 'http://api.timezonedb.com/v2.1/get-time-zone?key=XH0FAMNCPCSD&format=json&by=position&' + "lat=" + generalData.value.lat + "&lng=" + generalData.value.lon;
    console.log(endpoint3)
    var xhr3 = new XMLHttpRequest();
    xhr3.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            var response = JSON.parse(this.responseText);
            if (response.status !== 'OK') {
                console.log('Sunrise/-fall query failed!');
                return
            }
            console.log(response)
            timeData.value = response;
            lookupLocalWeather();
        }
    };
    xhr3.open('GET', endpoint3, true);
    xhr3.send();
}

function lookupLocalWeather() {
    //API Key: XH0FAMNCPCSD

    var endpoint4 = 'https://api.open-meteo.com/v1/forecast?&' + "latitude=" + generalData.value.lat + "&longitude=" + generalData.value.lon + '&current=temperature_2m,wind_speed_10m';
    console.log(endpoint4)
    var xhr4 = new XMLHttpRequest();
    xhr4.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            var response = JSON.parse(this.responseText);
            weatherData.value = response;
            dataReceived.value = true;
        }
    };
    xhr4.open('GET', endpoint4, true);
    xhr4.send();
}

</script>

<template>
    <h1>IP Adress Lookup</h1>
    <form @submit.prevent="lookupInput">
        <span>
            <input id="userInput" v-model="userInput" type="text" placeholder="URL or IP">
        </span>
        <br>
        <button>Search</button>
    </form>
    
    <div v-if="dataReceived">
        <h2>Result:</h2>
        <p>IP Adress: {{ generalData.query }}</p>
        <p>Country: {{ generalData.country }}</p>
        <p>City: {{ generalData.city }}</p>
        <p>Geo-coordinates: latitude: {{ generalData.lat }}, longitude: {{ generalData.lon }}</p>
        <p>Local time: {{ timeData.formatted }}</p>
        <p>Timezone: {{ generalData.timezone }}</p>
        <p>Time Sunrise: {{ sunriseData.results.sunrise }}</p>
        <p>Time Sunset: {{ sunriseData.results.sunset }}</p>
        <p>Current weather: {{ weatherData.current.temperature_2m + weatherData.current_units.temperature_2m + " " +
            weatherData.current.wind_speed_10m + weatherData.current_units.wind_speed_10m }}</p>
    </div>
    <div>
        <p style="color: red">{{ errormessage }}</p>
    </div>
</template>
