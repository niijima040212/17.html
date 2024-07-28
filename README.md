<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>サイクリング行先推薦アプリ</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container {
            width: 80%;
            margin: 20px auto;
            text-align: center;
        }
        .button-container, .elevation-container, .start-container {
            margin: 20px 0;
        }
        .button-container button, .elevation-container button, .start-container button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
        }
        .map {
            width: 100%;
            height: 500px;
            background-color: #e0e0e0;
            margin: 20px 0;
            position: relative;
        }
        .results {
            display: flex;
            justify-content: space-around;
            width: 100%;
            margin: 20px 0;
        }
        .results div {
            width: 30%;
        }
        .toggle-container {
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>スポット検索エンジン</h1>
        <div class="start-container">
            <button onclick="selectStart('武蔵野大学有明キャンパス')">武蔵野大学有明キャンパス</button>
            <button onclick="selectStart('武蔵野大学武蔵野キャンパス')">武蔵野大学武蔵野キャンパス</button>
        </div>
        <div class="button-container">
            <button onclick="selectDistance(10)">10km以上</button>
            <button onclick="selectDistance(40)">40km以上</button>
            <button onclick="selectDistance(80)">80km以上</button>
            <button onclick="selectDistance(100)">100km以上</button>
        </div>
        <div class="elevation-container">
            <button onclick="selectElevation('any')">指定しない</button>
            <button onclick="selectElevation('flat')">平坦</button>
            <button onclick="selectElevation('hilly')">高め</button>
        </div>
        <div>
            <button onclick="recommendSpot()">スポットを推薦</button>
        </div>
        <div class="toggle-container">
            <label>
                <input type="radio" name="difficulty" value="beginner" onchange="recommendForDifficulty()"> 初心者
            </label>
            <label>
                <input type="radio" name="difficulty" value="advanced" onchange="recommendForDifficulty()"> 上級者
            </label>
        </div>
        <div class="map" id="map"></div>
        <div class="results">
            <div>合計距離: <span id="totalDistance"></span> km <br> スポット: <span id="selectedSpot"></span></div>
            <div>獲得標高: <span id="totalElevation"></span></div>
        </div>
    </div>

    <script>
        let map, directionsService, directionsRenderer, infowindow, distanceMatrixService;

        // スポットの緯度と経度
        const spots = {
            '江の島': { lat: 35.2995, lng: 139.4708 },
            '川越駅': { lat: 35.9078, lng: 139.4823 },
            '明治神宮': { lat: 35.6764, lng: 139.6993 },
            '等々力公園': { lat: 35.6067, lng: 139.6485 },
            '上野恩賜公園': { lat: 35.7156, lng: 139.7735 },
            '昭和記念公園': { lat: 35.7046, lng: 139.4078 },
            '多摩湖': { lat: 35.7403, lng: 139.4293 },
            '東京タワー': { lat: 35.6586, lng: 139.7454 },
            '浅草寺': { lat: 35.7148, lng: 139.7967 },
            'お台場海浜公園': { lat: 35.6286, lng: 139.7766 },
            '代々木公園': { lat: 35.6717, lng: 139.6945 },
            '六本木ヒルズ': { lat: 35.6604, lng: 139.7292 },
            '築地市場': { lat: 35.6655, lng: 139.7708 },
            '銀座': { lat: 35.6719, lng: 139.7644 },
            '新宿御苑': { lat: 35.6852, lng: 139.7100 },
            '秋葉原': { lat: 35.6987, lng: 139.7731 },
            '東京スカイツリー': { lat: 35.7100, lng: 139.8107 },
            '靖国神社': { lat: 35.6940, lng: 139.7433 },
            '東京国立博物館': { lat: 35.7188, lng: 139.7765 },
            // Additional spots
            '箱根': { lat: 35.2329, lng: 139.1068 },
            '鎌倉': { lat: 35.3195, lng: 139.5505 },
            '湘南': { lat: 35.3333, lng: 139.3420 },
            '大宮公園': { lat: 35.9061, lng: 139.6266 },
            '成田山新勝寺': { lat: 35.7750, lng: 140.3188 },
            '葛西臨海公園': { lat: 35.6445, lng: 139.8630 },
            '日光東照宮': { lat: 36.7578, lng: 139.5984 },
            '宇都宮二荒山神社': { lat: 36.5551, lng: 139.8828 },
            '富士山': { lat: 35.3606, lng: 138.7274 },
            '箱根湯本': { lat: 35.2330, lng: 139.1055 },
            '河口湖': { lat: 35.4913, lng: 138.7559 },
            '山中湖': { lat: 35.4152, lng: 138.8723 },
            '松本城': { lat: 36.2381, lng: 137.9704 },
            '軽井沢': { lat: 36.3483, lng: 138.5983 },
            '那須高原': { lat: 37.1231, lng: 139.9631 },
            '草津温泉': { lat: 36.6180, lng: 138.5996 },
            '妙高高原': { lat: 36.9372, lng: 138.1971 },
            '白馬': { lat: 36.6985, lng: 137.8597 },
            '蓼科': { lat: 36.0581, lng: 138.2570 },
            '小淵沢': { lat: 35.8952, lng: 138.2861 },
            '八ヶ岳': { lat: 35.9639, lng: 138.2760 },
            '上高地': { lat: 36.2604, lng: 137.6416 },
            '御殿場': { lat: 35.3086, lng: 138.9358 },
            '伊豆高原': { lat: 34.8958, lng: 139.1255 },
            '熱海': { lat: 35.0965, lng: 139.0770 }
        };

        const startLocations = {
            '武蔵野大学有明キャンパス': { lat: 35.6342, lng: 139.7944 },
            '武蔵野大学武蔵野キャンパス': { lat: 35.7061, lng: 139.5406 }
        };

        let currentDifficulty = null;
        let selectedDistance = null;
        let selectedElevation = null;
        let startLocation = null;

        function initMap() {
            map = new google.maps.Map(document.getElementById('map'), {
                zoom: 10,
                center: { lat: 35.6895, lng: 139.6917 } // 東京の中心
            });
            directionsService = new google.maps.DirectionsService();
            directionsRenderer = new google.maps.DirectionsRenderer();
            directionsRenderer.setMap(map);
            infowindow = new google.maps.InfoWindow();
            distanceMatrixService = new google.maps.DistanceMatrixService();
        }

        function recommendForDifficulty() {
            const difficultyRadios = document.getElementsByName('difficulty');
            for (const radio of difficultyRadios) {
                if (radio.checked) {
                    currentDifficulty = radio.value;
                    console.log('Difficulty selected:', currentDifficulty);
                    if (startLocation && selectedDistance !== null && selectedElevation !== null) {
                        updateRouteForDifficulty();
                    }
                }
            }
        }

        function selectDistance(distance) {
            selectedDistance = distance;
            console.log('Distance selected:', selectedDistance);
        }

        function selectElevation(elevation) {
            selectedElevation = elevation;
            console.log('Elevation selected:', selectedElevation);
        }

        function selectStart(location) {
            startLocation = location;
            console.log('Start location selected:', startLocation);
        }

        function recommendSpot() {
            if (startLocation && selectedDistance !== null && selectedElevation !== null) {
                console.log('Recommending spot for:', startLocation, selectedDistance, selectedElevation);
                updateRoute();
            } else {
                alert("出発地点、距離、獲得標高を選択してください");
            }
        }

        function updateRoute() {
            const spotEntries = Object.entries(spots);
            const spotLocations = spotEntries.map(entry => entry[1]);
            const selectedSpots = spotEntries.map(entry => entry[0]);

            const chunkSize = 25;
            const spotChunks = [];
            for (let i = 0; i < spotLocations.length; i += chunkSize) {
                spotChunks.push({
                    spots: selectedSpots.slice(i, i + chunkSize),
                    locations: spotLocations.slice(i, i + chunkSize)
                });
            }

            let allDistances = [];
            let processedChunks = 0;

            spotChunks.forEach(chunk => {
                distanceMatrixService.getDistanceMatrix({
                    origins: [startLocations[startLocation]],
                    destinations: chunk.locations,
                    travelMode: google.maps.TravelMode.BICYCLING
                }, function(response, status) {
                    if (status === 'OK') {
                        const distances = response.rows[0].elements;
                        for (let i = 0; i < distances.length; i++) {
                            if (distances[i].status === 'OK') {
                                const distanceKm = distances[i].distance.value / 1000;
                                if (distanceKm >= selectedDistance) {
                                    allDistances.push({
                                        spot: chunk.spots[i],
                                        distance: distanceKm
                                    });
                                }
                            } else {
                                console.error('Distance calculation failed for spot:', chunk.spots[i], distances[i].status);
                            }
                        }
                    } else {
                        console.error('Distance Matrix failed due to ' + status);
                    }

                    processedChunks++;
                    if (processedChunks === spotChunks.length) {
                        allDistances.sort((a, b) => a.distance - b.distance);
                        const recommendedSpot = allDistances.length > 0 ? allDistances[0] : null;

                        if (recommendedSpot) {
                            document.getElementById('selectedSpot').innerText = recommendedSpot.spot;
                            document.getElementById('totalDistance').innerText = recommendedSpot.distance.toFixed(2);
                            displayMap([recommendedSpot.spot]);
                        } else {
                            document.getElementById('selectedSpot').innerText = "該当するスポットがありません";
                            document.getElementById('totalDistance').innerText = "0.00";
                        }
                    }
                });
            });
        }

        function updateRouteForDifficulty() {
            // 難易度に応じたルート推薦ロジックを実装
            updateRoute();
        }

        function displayMap(routeSpots) {
            const waypoints = routeSpots.map(spot => ({
                location: spots[spot],
                stopover: true
            }));

            const routeRequest = {
                origin: startLocations[startLocation],
                destination: waypoints.length > 0 ? waypoints[waypoints.length - 1].location : startLocations[startLocation],
                waypoints: waypoints.slice(0, waypoints.length - 1),
                travelMode: google.maps.TravelMode.BICYCLING
            };

            directionsService.route(routeRequest, (result, status) => {
                if (status === 'OK') {
                    directionsRenderer.setDirections(result);

                    // スポットにマーカーを表示し、スポット名を吹き出しで表示
                    for (let i = 0; i < routeSpots.length; i++) {
                        const marker = new google.maps.Marker({
                            position: spots[routeSpots[i]],
                            map: map,
                            title: routeSpots[i]
                        });
                        google.maps.event.addListener(marker, 'click', function() {
                            infowindow.setContent(routeSpots[i]);
                            infowindow.open(map, marker);
                        });
                    }
                } else {
                    console.error('Error fetching directions', status);
                }
            });
        }

        // 初期表示のために関数を呼び出す
        window.initMap = initMap;
    </script>
    <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBgr2mPV7F_Qeiez6f7gMXARW1yyJuyMAY&callback=initMap" async defer></script>
</body>
</html>
