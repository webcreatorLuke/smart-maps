<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Maps - Navigation & GPS</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            height: 100vh;
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 15px 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            position: relative;
            z-index: 1000;
        }

        .header h1 {
            font-size: 24px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .header-icon {
            width: 32px;
            height: 32px;
        }

        .container {
            display: flex;
            height: calc(100vh - 70px);
        }

        .sidebar {
            width: 380px;
            background: white;
            box-shadow: 2px 0 10px rgba(0,0,0,0.1);
            overflow-y: auto;
            z-index: 999;
        }

        .search-box {
            padding: 20px;
            border-bottom: 1px solid #e0e0e0;
        }

        .search-input {
            width: 100%;
            padding: 12px 15px;
            border: 2px solid #667eea;
            border-radius: 8px;
            font-size: 14px;
            margin-bottom: 10px;
            transition: all 0.3s;
        }

        .search-input:focus {
            outline: none;
            border-color: #764ba2;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }

        .btn {
            padding: 12px 20px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 600;
            transition: all 0.3s;
            width: 100%;
        }

        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }

        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
        }

        .btn-secondary {
            background: #f0f0f0;
            color: #333;
            margin-top: 10px;
        }

        .btn-secondary:hover {
            background: #e0e0e0;
        }

        .location-info {
            padding: 20px;
            border-bottom: 1px solid #e0e0e0;
        }

        .info-card {
            background: #f8f9fa;
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 15px;
        }

        .info-card h3 {
            color: #667eea;
            font-size: 14px;
            margin-bottom: 8px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .info-card p {
            color: #333;
            font-size: 16px;
            line-height: 1.6;
        }

        .route-options {
            padding: 20px;
        }

        .route-option {
            background: white;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 10px;
            cursor: pointer;
            transition: all 0.3s;
        }

        .route-option:hover {
            border-color: #667eea;
            box-shadow: 0 3px 10px rgba(102, 126, 234, 0.2);
        }

        .route-option.active {
            border-color: #667eea;
            background: #f8f9ff;
        }

        .route-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
        }

        .route-time {
            font-size: 18px;
            font-weight: bold;
            color: #667eea;
        }

        .route-distance {
            color: #666;
            font-size: 14px;
        }

        .route-details {
            color: #333;
            font-size: 13px;
        }

        .eta-badge {
            background: #4caf50;
            color: white;
            padding: 4px 10px;
            border-radius: 12px;
            font-size: 12px;
            font-weight: 600;
        }

        #map {
            flex: 1;
            height: 100%;
        }

        .map-controls {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .map-btn {
            background: white;
            border: none;
            width: 45px;
            height: 45px;
            border-radius: 50%;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
            transition: all 0.3s;
        }

        .map-btn:hover {
            background: #667eea;
            color: white;
            transform: scale(1.1);
        }

        .traffic-indicator {
            display: flex;
            gap: 8px;
            margin-top: 8px;
        }

        .traffic-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
        }

        .traffic-low {
            background: #4caf50;
        }

        .traffic-medium {
            background: #ff9800;
        }

        .traffic-high {
            background: #f44336;
        }

        .features-grid {
            padding: 20px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .feature-btn {
            padding: 15px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            background: white;
            cursor: pointer;
            transition: all 0.3s;
            text-align: center;
            font-size: 12px;
        }

        .feature-btn:hover {
            border-color: #667eea;
            background: #f8f9ff;
        }

        .feature-icon {
            font-size: 24px;
            margin-bottom: 5px;
        }

        .status-bar {
            background: #f8f9fa;
            padding: 10px 20px;
            border-bottom: 1px solid #e0e0e0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 12px;
        }

        .status-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .status-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: #4caf50;
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }

        .loading {
            display: none;
            text-align: center;
            padding: 20px;
            color: #667eea;
        }

        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .direction-steps {
            padding: 20px;
            max-height: 300px;
            overflow-y: auto;
        }

        .direction-step {
            background: #f8f9fa;
            border-left: 3px solid #667eea;
            padding: 12px;
            margin-bottom: 10px;
            border-radius: 5px;
        }

        .step-number {
            font-weight: bold;
            color: #667eea;
            margin-right: 8px;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>
            <svg class="header-icon" viewBox="0 0 24 24" fill="white">
                <path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/>
            </svg>
            Smart Maps
        </h1>
    </div>

    <div class="container">
        <div class="sidebar">
            <div class="status-bar">
                <div class="status-item">
                    <div class="status-dot"></div>
                    <span>GPS Active</span>
                </div>
                <div class="status-item">
                    <span id="currentTime"></span>
                </div>
            </div>

            <div class="search-box">
                <input type="text" id="startLocation" class="search-input" placeholder="📍 Your location (auto-detected)">
                <input type="text" id="destination" class="search-input" placeholder="🔍 Where to?">
                <button class="btn btn-primary" onclick="getDirections()">Get Directions</button>
                <button class="btn btn-secondary" onclick="findMyLocation()">📍 Use My Current Location</button>
            </div>

            <div class="loading" id="loading">
                <div class="spinner"></div>
                <p>Finding best route...</p>
            </div>

            <div class="location-info" id="locationInfo" style="display: none;">
                <div class="info-card">
                    <h3>Current Location</h3>
                    <p id="currentLocationText">Detecting...</p>
                </div>
                <div class="info-card">
                    <h3>Coordinates</h3>
                    <p id="coordinates">--</p>
                </div>
                <div class="info-card">
                    <h3>Speed</h3>
                    <p id="speed">-- mph</p>
                </div>
            </div>

            <div class="route-options" id="routeOptions" style="display: none;">
                <h3 style="margin-bottom: 15px; color: #333;">Route Options</h3>
                <div class="route-option active" onclick="selectRoute(0)">
                    <div class="route-header">
                        <div class="route-time">25 min</div>
                        <div class="eta-badge">Fastest</div>
                    </div>
                    <div class="route-distance">12.3 miles via Highway 101</div>
                    <div class="route-details">Usually less traffic on this route</div>
                    <div class="traffic-indicator">
                        <div class="traffic-dot traffic-low"></div>
                        <div class="traffic-dot traffic-low"></div>
                        <div class="traffic-dot traffic-low"></div>
                        <span style="font-size: 11px; color: #4caf50;">Light traffic</span>
                    </div>
                </div>
                <div class="route-option" onclick="selectRoute(1)">
                    <div class="route-header">
                        <div class="route-time">32 min</div>
                        <div class="eta-badge" style="background: #ff9800;">Scenic</div>
                    </div>
                    <div class="route-distance">10.8 miles via Coastal Road</div>
                    <div class="route-details">Beautiful views, fewer highways</div>
                    <div class="traffic-indicator">
                        <div class="traffic-dot traffic-medium"></div>
                        <div class="traffic-dot traffic-medium"></div>
                        <div class="traffic-dot traffic-low"></div>
                        <span style="font-size: 11px; color: #ff9800;">Moderate traffic</span>
                    </div>
                </div>
                <div class="route-option" onclick="selectRoute(2)">
                    <div class="route-header">
                        <div class="route-time">28 min</div>
                        <div class="eta-badge" style="background: #2196f3;">Avoid Tolls</div>
                    </div>
                    <div class="route-distance">13.1 miles via Main Street</div>
                    <div class="route-details">No toll roads on this route</div>
                    <div class="traffic-indicator">
                        <div class="traffic-dot traffic-low"></div>
                        <div class="traffic-dot traffic-medium"></div>
                        <div class="traffic-dot traffic-low"></div>
                        <span style="font-size: 11px; color: #666;">Normal traffic</span>
                    </div>
                </div>
            </div>

            <div class="direction-steps" id="directionSteps" style="display: none;">
                <h3 style="margin-bottom: 15px; color: #333;">Turn-by-Turn Directions</h3>
                <div class="direction-step">
                    <span class="step-number">1.</span>
                    Head north on Main St toward 1st Ave
                    <div style="font-size: 12px; color: #666; margin-top: 5px;">0.3 miles</div>
                </div>
                <div class="direction-step">
                    <span class="step-number">2.</span>
                    Turn right onto Highway 101 N
                    <div style="font-size: 12px; color: #666; margin-top: 5px;">8.5 miles</div>
                </div>
                <div class="direction-step">
                    <span class="step-number">3.</span>
                    Take exit 42B for Downtown
                    <div style="font-size: 12px; color: #666; margin-top: 5px;">0.5 miles</div>
                </div>
                <div class="direction-step">
                    <span class="step-number">4.</span>
                    Continue straight onto Market St
                    <div style="font-size: 12px; color: #666; margin-top: 5px;">2.1 miles</div>
                </div>
                <div class="direction-step">
                    <span class="step-number">5.</span>
                    Your destination will be on the right
                    <div style="font-size: 12px; color: #666; margin-top: 5px;">0.9 miles</div>
                </div>
            </div>

            <div class="features-grid">
                <div class="feature-btn" onclick="toggleTraffic()">
                    <div class="feature-icon">🚦</div>
                    <div>Live Traffic</div>
                </div>
                <div class="feature-btn" onclick="toggleSatellite()">
                    <div class="feature-icon">🛰️</div>
                    <div>Satellite View</div>
                </div>
                <div class="feature-btn" onclick="findNearby('gas')">
                    <div class="feature-icon">⛽</div>
                    <div>Gas Stations</div>
                </div>
                <div class="feature-btn" onclick="findNearby('food')">
                    <div class="feature-icon">🍔</div>
                    <div>Restaurants</div>
                </div>
                <div class="feature-btn" onclick="findNearby('parking')">
                    <div class="feature-icon">🅿️</div>
                    <div>Parking</div>
                </div>
                <div class="feature-btn" onclick="findNearby('hotel')">
                    <div class="feature-icon">🏨</div>
                    <div>Hotels</div>
                </div>
                <div class="feature-btn" onclick="saveLocation()">
                    <div class="feature-icon">⭐</div>
                    <div>Save Place</div>
                </div>
                <div class="feature-btn" onclick="shareLocation()">
                    <div class="feature-icon">📤</div>
                    <div>Share</div>
                </div>
            </div>
        </div>

        <div style="position: relative; flex: 1;">
            <div id="map"></div>
            <div class="map-controls">
                <button class="map-btn" onclick="zoomIn()" title="Zoom In">+</button>
                <button class="map-btn" onclick="zoomOut()" title="Zoom Out">−</button>
                <button class="map-btn" onclick="recenterMap()" title="My Location">📍</button>
                <button class="map-btn" onclick="toggle3D()" title="3D View">🗺️</button>
            </div>
        </div>
    </div>

    <script>
        let map;
        let userMarker;
        let destinationMarker;
        let currentPosition = null;
        let watchId = null;
        let routeLine = null;

        // Initialize map
        function initMap() {
            map = L.map('map').setView([37.7749, -122.4194], 13);
            
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap contributors',
                maxZoom: 19
            }).addTo(map);

            findMyLocation();
        }

        // Update time
        function updateTime() {
            const now = new Date();
            document.getElementById('currentTime').textContent = now.toLocaleTimeString();
        }
        setInterval(updateTime, 1000);
        updateTime();

        // Find user's current location
        function findMyLocation() {
            if (navigator.geolocation) {
                document.getElementById('loading').style.display = 'block';
                
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        currentPosition = {
                            lat: position.coords.latitude,
                            lng: position.coords.longitude
                        };
                        
                        map.setView([currentPosition.lat, currentPosition.lng], 15);
                        
                        if (userMarker) {
                            map.removeLayer(userMarker);
                        }
                        
                        const blueIcon = L.divIcon({
                            className: 'custom-marker',
                            html: '<div style="background: #4285f4; width: 20px; height: 20px; border-radius: 50%; border: 3px solid white; box-shadow: 0 2px 5px rgba(0,0,0,0.3);"></div>',
                            iconSize: [20, 20]
                        });
                        
                        userMarker = L.marker([currentPosition.lat, currentPosition.lng], { icon: blueIcon })
                            .addTo(map)
                            .bindPopup('You are here!')
                            .openPopup();
                        
                        document.getElementById('locationInfo').style.display = 'block';
                        document.getElementById('currentLocationText').textContent = 'Location detected successfully';
                        document.getElementById('coordinates').textContent = 
                            `${currentPosition.lat.toFixed(6)}, ${currentPosition.lng.toFixed(6)}`;
                        
                        // Watch position for real-time updates
                        if (watchId) {
                            navigator.geolocation.clearWatch(watchId);
                        }
                        watchId = navigator.geolocation.watchPosition(updatePosition);
                        
                        document.getElementById('loading').style.display = 'none';
                        
                        document.getElementById('startLocation').value = 'Current Location';
                    },
                    (error) => {
                        document.getElementById('loading').style.display = 'none';
                        alert('Unable to get your location. Please enable location services.');
                        // Default to San Francisco
                        map.setView([37.7749, -122.4194], 13);
                    },
                    { enableHighAccuracy: true }
                );
            } else {
                alert('Geolocation is not supported by your browser.');
            }
        }

        // Update position in real-time
        function updatePosition(position) {
            currentPosition = {
                lat: position.coords.latitude,
                lng: position.coords.longitude
            };
            
            if (userMarker) {
                userMarker.setLatLng([currentPosition.lat, currentPosition.lng]);
            }
            
            document.getElementById('coordinates').textContent = 
                `${currentPosition.lat.toFixed(6)}, ${currentPosition.lng.toFixed(6)}`;
            
            if (position.coords.speed) {
                const speedMph = (position.coords.speed * 2.237).toFixed(1);
                document.getElementById('speed').textContent = `${speedMph} mph`;
            }
        }

        // Get directions
        function getDirections() {
            const destination = document.getElementById('destination').value;
            
            if (!destination) {
                alert('Please enter a destination');
                return;
            }
            
            if (!currentPosition) {
                alert('Please enable location services first');
                findMyLocation();
                return;
            }
            
            document.getElementById('loading').style.display = 'block';
            
            // Simulate getting directions
            setTimeout(() => {
                document.getElementById('loading').style.display = 'none';
                document.getElementById('routeOptions').style.display = 'block';
                document.getElementById('directionSteps').style.display = 'block';
                
                // Add destination marker
                const randomLat = currentPosition.lat + (Math.random() - 0.5) * 0.1;
                const randomLng = currentPosition.lng + (Math.random() - 0.5) * 0.1;
                
                if (destinationMarker) {
                    map.removeLayer(destinationMarker);
                }
                
                const redIcon = L.divIcon({
                    className: 'custom-marker',
                    html: '<div style="background: #ea4335; width: 30px; height: 30px; border-radius: 50% 50% 50% 0; transform: rotate(-45deg); border: 3px solid white; box-shadow: 0 2px 5px rgba(0,0,0,0.3);"><div style="background: white; width: 10px; height: 10px; border-radius: 50%; position: absolute; top: 7px; left: 7px;"></div></div>',
                    iconSize: [30, 30]
                });
                
                destinationMarker = L.marker([randomLat, randomLng], { icon: redIcon })
                    .addTo(map)
                    .bindPopup(destination);
                
                // Draw route line
                if (routeLine) {
                    map.removeLayer(routeLine);
                }
                
                routeLine = L.polyline([
                    [currentPosition.lat, currentPosition.lng],
                    [randomLat, randomLng]
                ], {
                    color: '#4285f4',
                    weight: 5,
                    opacity: 0.7
                }).addTo(map);
                
                // Fit map to show route
                map.fitBounds(routeLine.getBounds(), { padding: [50, 50] });
            }, 1500);
        }

        // Select route
        function selectRoute(index) {
            const options = document.querySelectorAll('.route-option');
            options.forEach((opt, i) => {
                if (i === index) {
                    opt.classList.add('active');
                } else {
                    opt.classList.remove('active');
                }
            });
        }

        // Map controls
        function zoomIn() {
            map.zoomIn();
        }

        function zoomOut() {
            map.zoomOut();
        }

        function recenterMap() {
            if (currentPosition) {
                map.setView([currentPosition.lat, currentPosition.lng], 15);
            } else {
                findMyLocation();
            }
        }

        function toggle3D() {
            alert('3D view feature - Would enable 3D buildings and terrain in a full implementation');
        }

        // Feature buttons
        function toggleTraffic() {
            alert('Live traffic overlay toggled! 🚦\nGreen = Light traffic\nYellow = Moderate traffic\nRed = Heavy traffic');
        }

        function toggleSatellite() {
            alert('Satellite view toggled! 🛰️\nIn a full version, this would switch between map and satellite imagery.');
        }

        function findNearby(type) {
            const types = {
                'gas': '⛽ Gas Stations',
                'food': '🍔 Restaurants',
                'parking': '🅿️ Parking Lots',
                'hotel': '🏨 Hotels'
            };
            alert(`Searching for nearby ${types[type]}...\nIn a full version, this would show all nearby ${types[type]} on the map.`);
        }

        function saveLocation() {
            if (currentPosition) {
                alert('📍 Location saved to favorites!\nCoordinates: ' + 
                    currentPosition.lat.toFixed(6) + ', ' + currentPosition.lng.toFixed(6));
            } else {
                alert('Please enable location services first');
            }
        }

        function shareLocation() {
            if (currentPosition) {
                const url = `https://maps.google.com/?q=${currentPosition.lat},${currentPosition.lng}`;
                navigator.clipboard.writeText(url).then(() => {
                    alert('📤 Location link copied to clipboard!\nShare this link with anyone.');
                });
            } else {
                alert('Please enable location services first');
            }
        }

        // Initialize on load
        window.onload = initMap;
    </script>
</body>
</html>
