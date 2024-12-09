<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Leaflet Map with Nearby Hospitals</title>
    
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    
    <style>
        #map {
            height: 600px; 
            width: 100%; 
        }
    </style>
</head>
<body>
    <h3>Nearby Hospitals Map</h3>
    <p id="status">Fetching your location...</p>
    
   
    <div id="map"></div>
    
   
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        
        const map = L.map('map').setView([20.5937, 78.9629], 5); // Default to India
        
        
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        }).addTo(map);

       
        const statusElement = document.getElementById("status");

       
        function fetchHospitals(lat, lon, radius = 5000) { 
            const overpassUrl = "https://overpass-api.de/api/interpreter";
            const overpassQuery = `
                [out:json];
                node["amenity"="hospital"](around:${radius},${lat},${lon});
                out body;
            `;

            fetch(`${overpassUrl}?data=${encodeURIComponent(overpassQuery)}`)
                .then(response => response.json())
                .then(data => {
                    if (data.elements.length === 0) {
                        alert("No hospitals found nearby!");
                        return;
                    }

                    
                    data.elements.forEach(hospital => {
                        if (hospital.lat && hospital.lon) {
                            const name = hospital.tags.name || "Unnamed Hospital";
                            const address = hospital.tags["addr:full"] || "Address not available";
                            const phone = hospital.tags.phone || "Phone not available";

                            const marker = L.marker([hospital.lat, hospital.lon]).addTo(map);
                            marker.bindPopup(`
                                <b>${name}</b><br>
                                Address: ${address}<br>
                                Phone: ${phone}<br>
                                <a href="#" onclick="showDetails('${name}', '${address}', '${phone}')">More Details</a>
                            `);
                        }
                    });

                    map.setView([lat, lon], 13);
                })
                .catch(error => {
                    console.error("Error fetching hospital data:", error);
                    alert("Failed to load nearby hospitals. Please try again later.");
                });
        }

        
        function onLocationSuccess(position) {
            const { latitude, longitude } = position.coords;

            
            map.setView([latitude, longitude], 13);
            statusElement.textContent = "Showing hospitals near your location.";

            
            fetchHospitals(latitude, longitude);
        }

        
        function onLocationError(error) {
            console.error("Geolocation error:", error.message);
            statusElement.textContent = "Unable to fetch your location. Please check your browser settings.";
        }

        
        function showDetails(name, address, phone) {
            alert(`Hospital Details:\n\nName: ${name}\nAddress: ${address}\nPhone: ${phone}`);
        }

        
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(onLocationSuccess, onLocationError);
        } else {
            statusElement.textContent = "Geolocation is not supported by your browser.";
        }
    </script>
</body>
</html>
