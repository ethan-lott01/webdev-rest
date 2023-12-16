<script setup>
import { reactive, ref, onMounted } from 'vue'

let crime_url = ref('');
//let coordinates = [44.955139,-93.102222];
let dialog_err = ref(false);
let map = reactive(
    {
        leaflet: null,
        center: {
            lat: 44.955139,
            lng: -93.102222,
            address: ''
        },
        zoom: 12,
        bounds: {
            nw: {lat: 45.008206, lng: -93.217977},
            se: {lat: 44.883658, lng: -92.993787}
        },
        neighborhood_markers: [
            {location: [44.942068, -93.020521], marker: null},
            {location: [44.977413, -93.025156], marker: null},
            {location: [44.931244, -93.079578], marker: null},
            {location: [44.956192, -93.060189], marker: null},
            {location: [44.978883, -93.068163], marker: null},
            {location: [44.975766, -93.113887], marker: null},
            {location: [44.959639, -93.121271], marker: null},
            {location: [44.947700, -93.128505], marker: null},
            {location: [44.930276, -93.119911], marker: null},
            {location: [44.982752, -93.147910], marker: null},
            {location: [44.963631, -93.167548], marker: null},
            {location: [44.973971, -93.197965], marker: null},
            {location: [44.949043, -93.178261], marker: null},
            {location: [44.934848, -93.176736], marker: null},
            {location: [44.913106, -93.170779], marker: null},
            {location: [44.937705, -93.136997], marker: null},
            {location: [44.949203, -93.093739], marker: null}
        ]
    });
let coordinates = [map.center.lat,map.center.lng];

// Vue callback for once <template> HTML has been added to web page
onMounted(() => {
    // Create Leaflet map (set bounds and valied zoom levels)
    map.leaflet = L.map('leafletmap').setView([map.center.lat, map.center.lng], map.zoom);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        minZoom: 11,
        maxZoom: 18
    }).addTo(map.leaflet);
    map.leaflet.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);

    // Add event listener for dragend
    map.leaflet.on('dragend', (e) => {
        // Update coordinates based on new center position
        coordinates[0] = e.target.getCenter().lat;
        coordinates[1] = e.target.getCenter().lng;

        // Update input field values with a slight delay (optional)
        setTimeout(() => {
            document.getElementById("latitude-input").value = coordinates[0];
            document.getElementById("longitude-input").value = coordinates[1];
        }, 100); // Replace 100 with your desired delay in milliseconds
    });

    // Get boundaries for St. Paul neighborhoods
    let district_boundary = new L.geoJson();
    district_boundary.addTo(map.leaflet);
    fetch('data/StPaulDistrictCouncil.geojson')
    .then((response) => {
        return response.json();
    })
    .then((result) => {
        result.features.forEach((value) => {
            district_boundary.addData(value);
        });
    })
    .catch((error) => {
        console.log('Error:', error);
    });
});


// FUNCTIONS
// Function called once user has entered REST API URL
function initializeCrimes() {
    // TODO: get code and neighborhood data
    //       get initial 1000 crimes
    // Get the base URL from the user input
    const baseUrl = crime_url.value;
    // Build the specific endpoint with coordinates (replace with your actual API syntax)
    //const endpoint = `${baseUrl}/crimedata?coordinates=${coordinates}`;
    // Use the built endpoint in your fetch call
    //fetch(endpoint, {
        //method: "POST",
        //body: JSON.stringify({}), // Replace with relevant data if needed
    //})
    //.then((response) => response.json())
    // ... Process the received data ...

}

// Function called when user presses 'OK' on dialog box
function closeDialog() {
    let dialog = document.getElementById('rest-dialog');
    let url_input = document.getElementById('dialog-url');
    if (crime_url.value !== '' && url_input.checkValidity()) {
        dialog_err.value = false;
        dialog.close();
        initializeCrimes();
    }
    else {
        dialog_err.value = true;
    }
}

// Function that submits location from input box
function submitLocation() {
    if (!coordinates) {
        console.error("Missing location data. Please enter all coordinates.");
        return;
    }
  // Send the coordinates to the REST API (replace with your actual API call)
  //fetch("/api/crimedata", {
    //method: "POST",
    //body: JSON.stringify({ coordinates }),
  //})
    //.then((response) => response.json())
    //.then((data) => {
      // Update map based on the received data (e.g., center on location, display markers)
      // ...

      //console.log("Location submitted successfully!");
    //})
    //.catch((error) => {
      //console.error("Error submitting location:", error);
    //});

    // Extract coordinates from the array
    let lat = coordinates[0];
    let lng = coordinates[1];
    
    // Update the map center
    map.leaflet.setView([lat, lng]);

    // Update the coordinates displayed
    coordinates[0] = lat;
    coordinates[1] = lng;
}

</script>

<template>
    <dialog id="rest-dialog" open>
        <h1 class="dialog-header">St. Paul Crime REST API</h1>
        <label class="dialog-label">URL: </label>
        <input id="dialog-url" class="dialog-input" type="url" v-model="crime_url" placeholder="http://localhost:8000" />
        <p class="dialog-error" v-if="dialog_err">Error: must enter valid URL</p>
        <br/>
        <button class="button" type="button" @click="closeDialog">OK</button>
    </dialog>
    <div class="grid-container ">
        <div class="grid-x grid-padding-x">
            <div id="leafletmap" class="cell auto"></div>
        </div>
        <div class="grid-x grid-padding-x">
            <div id="input-box" class="cell auto">
                <div>Enter Location: {{ coordinates }}</div>
                <input id="lat" v-model="coordinates[0]" placeholder="Latitude"/>
                <input id="long" v-model="coordinates[1]" placeholder="Longitude"/>
                <input id="addy" v-model="addy" placeholder="Address"/>
                <button v-on:click="submitLocation()">Go</button>
            </div>
        </div>
    </div>
</template>

<style>
#rest-dialog {
    width: 20rem;
    margin-top: 1rem;
    z-index: 1000;
}

#leafletmap {
    height: 500px;
}

.dialog-header {
    font-size: 1.2rem;
    font-weight: bold;
}

.dialog-label {
    font-size: 1rem;
}

.dialog-input {
    font-size: 1rem;
    width: 100%;
}

.dialog-error {
    font-size: 1rem;
    color: #D32323;
}
</style>
