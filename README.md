<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clothing Matcher App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            background: linear-gradient(to bottom, green, blue);
            color: white;
        }
        .image-container {
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 20px;
        }
        .image-container img {
            margin: 0 10px;
            width: 200px;
            height: auto;
            border: 1px solid #ccc;
        }
        button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
        #results, #library-page, #matches-page {
            display: none;
        }
        #popup {
            display: none;
            font-size: 18px;
            color: white;
            background: rgba(0, 0, 0, 0.8);
            padding: 20px;
            border-radius: 10px;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 1000;
        }
    </style>
</head>
<body>
    <h1>Clothing Matcher</h1>
    <div>
        <label for="shirts">Upload Shirts:</label>
        <input type="file" id="shirts" multiple accept="image/*"><br><br>
        <label for="pants">Upload Pants:</label>
        <input type="file" id="pants" multiple accept="image/*"><br><br>
    </div>

    <div class="image-container">
        <img id="current-shirt" src="" alt="Current Shirt">
        <img id="current-pants" src="" alt="Current Pants">
    </div>
    <div>
        <button onclick="startMatching()">Start Matching</button>
        <button onclick="swipeLeft()">Swipe Left (No Match)</button>
        <button onclick="swipeRight()">Swipe Right (Match)</button>
        <button onclick="exportMatches()">Export Matches</button>
        <button onclick="importMatches()">Import Matches</button>
        <button onclick="resetMatches()">Reset Matches</button>
        <button onclick="showLibrary()">View Library</button>
    </div>

    <div id="results">
        <h2>Unmatched Items</h2>
        <div id="unmatched-shirts"></div>
        <div id="unmatched-pants"></div>
    </div>

    <div id="popup">Last match complete! Click View Library.</div>

    <div id="library-page">
        <h2>Library</h2>
        <div id="matches-list"></div>
        <button onclick="backToMain()">Back</button>
    </div>

    <div id="matches-page">
        <h2>Matches</h2>
        <div id="matches-details"></div>
        <button onclick="backToLibrary()">Back</button>
    </div>

    <script>
        let shirts = [];
        let pants = [];
        let currentShirtIndex = 0;
        let currentPantsIndex = 0;
        let matches = [];

        document.getElementById('shirts').addEventListener('change', async (event) => {
            shirts = await Promise.all(Array.from(event.target.files).map(file => fileToBase64(file)));
            alert('Shirts uploaded successfully!');
        });

        document.getElementById('pants').addEventListener('change', async (event) => {
            pants = await Promise.all(Array.from(event.target.files).map(file => fileToBase64(file)));
            alert('Pants uploaded successfully!');
        });

        async function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
                reader.readAsDataURL(file);
            });
        }

        function startMatching() {
            if (shirts.length === 0 || pants.length === 0) {
                alert('Please upload both shirts and pants to start matching.');
                return;
            }
            currentShirtIndex = 0;
            currentPantsIndex = 0;
            loadImages();
        }

        function loadImages() {
            if (currentShirtIndex < shirts.length && currentPantsIndex < pants.length) {
                document.getElementById('current-shirt').src = shirts[currentShirtIndex];
                document.getElementById('current-pants').src = pants[currentPantsIndex];
            } else if (currentShirtIndex < shirts.length) {
                currentPantsIndex = 0;
                currentShirtIndex++;
                loadImages();
            } else {
                document.getElementById('popup').innerHTML = 'Last match complete! Click View Library.';
                document.getElementById('popup').style.display = 'block';
            }
        }

        function swipeLeft() {
            currentPantsIndex++;
            if (currentPantsIndex >= pants.length) {
                currentPantsIndex = 0;
                currentShirtIndex++;
            }
            loadImages();
        }

        function swipeRight() {
            const match = matches.find(m => m.shirt === shirts[currentShirtIndex]);
            if (match) {
                match.pants.push(pants[currentPantsIndex]);
            } else {
                matches.push({ shirt: shirts[currentShirtIndex], pants: [pants[currentPantsIndex]] });
            }
            currentPantsIndex++;
            if (currentPantsIndex >= pants.length) {
                currentPantsIndex = 0;
                currentShirtIndex++;
            }
            loadImages();
        }

        function exportMatches() {
            const data = JSON.stringify(matches, null, 2);
            const blob = new Blob([data], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = url;
            link.download = 'matches.json';
            link.click();
            URL.revokeObjectURL(url);
        }

        function importMatches() {
            const input = document.createElement('input');
            input.type = 'file';
            input.accept = 'application/json';
            input.onchange = (event) => {
                const file = event.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        matches = JSON.parse(e.target.result);
                        document.getElementById('popup').innerHTML = 'Matches imported! Click View Library.';
                        document.getElementById('popup').style.display = 'block';
                    };
                    reader.readAsText(file);
                }
            };
            input.click();
        }

        function resetMatches() {
            matches = [];
            alert('Matches have been reset!');
        }

        function showLibrary() {
            const matchesList = document.getElementById('matches-list');
            matchesList.innerHTML = '';
            matches.forEach(match => {
                const shirtImg = document.createElement('img');
                shirtImg.src = match.shirt;
                shirtImg.style.cursor = 'pointer';
                shirtImg.onclick = () => showMatchDetails(match);
                matchesList.appendChild(shirtImg);
            });
            document.getElementById('popup').style.display = 'none';
            document.getElementById('library-page').style.display = 'block';
        }

        function showMatchDetails(match) {
            const matchesDetails = document.getElementById('matches-details');
            matchesDetails.innerHTML = `<h3>Pants matched with this shirt:</h3><img src="${match.shirt}" alt="Shirt"><br>`;
            match.pants.forEach(pants => {
                const pantsImg = document.createElement('img');
                pantsImg.src = pants;
                pantsImg.style.width = "200px";
                pantsImg.style.height = "auto";
                matchesDetails.appendChild(pantsImg);
            });
            document.getElementById('matches-page').style.display = 'block';
            document.getElementById('library-page').style.display = 'none';
        }

        function backToMain() {
            document.getElementById('library-page').style.display = 'none';
        }

        function backToLibrary() {
            document.getElementById('matches-page').style.display = 'none';
            document.getElementById('library-page').style.display = 'block';
        }
    </script>
</body>
</html>
