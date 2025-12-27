<script>
  import { onMount } from 'svelte';
  import 'leaflet/dist/leaflet.css';
  import L from 'leaflet';

  let map;
  let lat = 51.505; // Default latitude
  let lng = -0.09;  // Default longitude
  let isLocationFetched = false; // Track if location is fetched

  // Function to get current geolocation
  function getCurrentLocation() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(position => {
        lat = position.coords.latitude;
        lng = position.coords.longitude;
        isLocationFetched = true; // Set the flag to true
        initializeMap(); // Initialize the map with current location
      }, error => {
        console.error("Error fetching location: ", error);
        alert('Unable to retrieve your location. Please ensure location services are enabled.');
      });
    } else {
      alert("Geolocation is not supported by this browser.");
    }
  }

  // Initialize the map
  function initializeMap() {
    if (map) {
      map.setView([lat, lng], 13); // Update map center
    } else {
      map = L.map('map').setView([lat, lng], 13); // Create map
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '© OpenStreetMap'
      }).addTo(map);
    }

    // Add a marker for the current location
    L.marker([lat, lng]).addTo(map)
      .bindPopup('Your Current Location!')
      .openPopup();
  }

  onMount(() => {
    getCurrentLocation(); // Fetch the location when the component mounts
  });
</script>

<main>
  
  <div id="map" style="height: 200px; width: 200px;"></div>
  <button class="normalButton1" on:click={getCurrentLocation}>Refresh Location</button> <!-- Optional refresh button -->
</main>

<style>
  main {
    background-color: black;
    color: white;
    padding: 20px;
  }

  #map {
    border: 1px solid white;
    margin-top:100px;
  }
  button{
    margin-left:30px;
    margin-top: 20px;
  }

</style>
