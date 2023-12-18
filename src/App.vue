<script setup>
import { reactive, ref, onMounted } from "vue";

let crime_url = ref("");
let dialog_err = ref(false);

let case_number = ref("");
let date = ref("");
let time = ref("");
let code = ref("");
let incident = ref("");
let police_grid = ref("");
let neighborhood_number = ref("");
let block = ref("");

let map = reactive({
  leaflet: null,
  center: {
    lat: 44.955139,
    lng: -93.102222,
    address: "",
  },
  zoom: 12,
  bounds: {
    nw: { lat: 45.008206, lng: -93.217977 },
    se: { lat: 44.883658, lng: -92.993787 },
  },
  neighborhood_markers: [
    { location: [44.942068, -93.020521], marker: null },
    { location: [44.977413, -93.025156], marker: null },
    { location: [44.931244, -93.079578], marker: null },
    { location: [44.956192, -93.060189], marker: null },
    { location: [44.978883, -93.068163], marker: null },
    { location: [44.975766, -93.113887], marker: null },
    { location: [44.959639, -93.121271], marker: null },
    { location: [44.9477, -93.128505], marker: null },
    { location: [44.930276, -93.119911], marker: null },
    { location: [44.982752, -93.14791], marker: null },
    { location: [44.963631, -93.167548], marker: null },
    { location: [44.973971, -93.197965], marker: null },
    { location: [44.949043, -93.178261], marker: null },
    { location: [44.934848, -93.176736], marker: null },
    { location: [44.913106, -93.170779], marker: null },
    { location: [44.937705, -93.136997], marker: null },
    { location: [44.949203, -93.093739], marker: null },
  ],
});

// Vue callback for once <template> HTML has been added to web page
onMounted(() => {
  // Create Leaflet map (set bounds and valid zoom levels)
  map.leaflet = L.map("leafletmap").setView(
    [map.center.lat, map.center.lng],
    map.zoom
  );
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    minZoom: 11,
    maxZoom: 18,
  }).addTo(map.leaflet);
  map.leaflet.setMaxBounds([
    [44.883658, -93.217977],
    [45.008206, -92.993787],
  ]);

  // Get boundaries for St. Paul neighborhoods
  let district_boundary = new L.geoJson();
  district_boundary.addTo(map.leaflet);
  fetch("data/StPaulDistrictCouncil.geojson")
    .then((response) => {
      return response.json();
    })
    .then((result) => {
      result.features.forEach((value) => {
        district_boundary.addData(value);
      });
    })
    .catch((error) => {
      console.log("Error:", error);
    });
});

// FUNCTIONS
// Function called once user has entered REST API URL
function initializeCrimes() {
  // TODO: get code and neighborhood data
  //       get initial 1000 crimes
}

// Function called when user presses 'OK' on dialog box
function closeDialog() {
  let dialog = document.getElementById("rest-dialog");
  let url_input = document.getElementById("dialog-url");
  if (crime_url.value !== "" && url_input.checkValidity()) {
    dialog_err.value = false;
    dialog.close();
    initializeCrimes();
  } else {
    dialog_err.value = true;
  }
}

async function submitForm(event) {
  try {
    // Check if form filled out
    if (
      !case_number._value ||
      !date._value ||
      !time._value ||
      !code._value ||
      !incident._value ||
      !police_grid._value ||
      !neighborhood_number._value ||
      !block._value
    ) {
      window.alert("Please fill out all form fields.");
      return;
    }
    const dateTimeString = `${date._value}T${time._value}:00`;

    const requestData = {
      case_number: case_number._value,
      date_time: dateTimeString,
      code: code._value,
      incident: incident._value,
      police_grid: police_grid._value,
      neighborhood_number: neighborhood_number._value,
      block: block._value,
    };

    console.log("Sending data to server:", requestData);

    const apiUrl = "http://localhost:8000/new-incident";

    // Add delay
    setTimeout(async () => {
      try {
        // Make PUT request using fetch
        console.log("Fetch Request:", apiUrl, JSON.stringify(requestData));
        const response = await fetch(apiUrl, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(requestData),
        });

        console.log("Server response:", response);

        // Handle the response
        if (response.ok) {
          console.log("Incident added successfully");
        } else {
          console.error("Failed to add incident:", response.statusText);
        }
      } catch (error) {
        console.error("Error:", error.message);
      }
    }, 2000);
  } catch (error) {
    console.error("Error:", error.message);
  }
}
</script>

<template>
  <dialog id="rest-dialog" open>
    <h1 class="dialog-header">St. Paul Crime REST API</h1>
    <label class="dialog-label">URL: </label>
    <input
      id="dialog-url"
      class="dialog-input"
      type="url"
      v-model="crime_url"
      placeholder="http://localhost:8000"
    />
    <p class="dialog-error" v-if="dialog_err">Error: must enter a valid URL</p>
    <br />
    <button class="button" type="button" @click="closeDialog">OK</button>
  </dialog>
  <div class="grid-container">
    <div class="grid-x grid-padding-x">
      <div id="leafletmap" class="cell auto"></div>
    </div>
  </div>

  <div><p>http://localhost:8000</p></div>

  <div style="width: 40%; margin-left: 20px">
    <form>
      <h2>Add New Incident</h2>
      <label for="case_number">Case Number:</label>
      <input type="text" id="case_number" v-model="case_number" />
      <label for="date">Date:</label>
      <input type="date" id="date" v-model="date" />
      <label for="time">Time:</label>
      <input type="time" id="time" v-model="time" />
      <label for="code">Code:</label>
      <input type="text" id="code" v-model="code" />
      <label for="incident">Incident:</label>
      <input type="text" id="incident" v-model="incident" />
      <label for="police_grid">Police Grid:</label>
      <input type="text" id="police_grid" v-model="police_grid" />
      <label for="neighborhood_number">Neighborhood Number:</label>
      <input
        type="text"
        id="neighborhood_number"
        v-model="neighborhood_number"
      />
      <label for="block">Address:</label>
      <input type="text" id="block" v-model="block" />
      <button class="button" @click="submitForm($event)">Add Incident</button>
    </form>
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
  color: #d32323;
}
</style>
