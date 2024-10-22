<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SpeakEasy</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f8f9fa;
            color: #333;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            text-align: center;
        }
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background-color: #ffffff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            max-width: 600px;
            margin: 20px;
        }
        h1 {
            font-size: 36px;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
            color: #007bff;
            font-style: italic;
            text-decoration: underline;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: #ffffff;
            border: none;
            border-radius: 5px;
            transition: background-color 0.3s;
            margin-top: 10px;
        }
        button:hover {
            background-color: #0056b3;
        }
        .page {
            display: none;
        }
        .history {
            margin-top: 20px;
            text-align: left;
        }
        .history-item {
            margin-bottom: 5px;
        }
    </style>
</head>
<body>
    <div class="container" id="landingPage">
        <h1>Welcome to <span class="title">SpeakEasy</span></h1>
        <button onclick="showPage('speechToTextPage')">Speech to Text</button>
        <button onclick="showPage('textToSpeechPage')">Text to Speech</button>
    </div>

    <div class="container page" id="speechToTextPage">
        <h1><span class="title">SpeakEasy</span></h1>
        <h2>Speech to Text</h2>
        <button id="startRecordingButton">Start Recording</button>
        <button id="stopRecordingButton" disabled>Stop Recording</button>
        <div>
            <label for="transcript">Transcript:</label>
            <textarea id="transcript" rows="10" cols="50" readonly></textarea>
        </div>
        <button id="clearTranscriptButton">Clear Text</button> <!-- Added Clear Text button -->
        <button onclick="showPage('landingPage')">Back to Home</button>
    </div>

    <div class="container page" id="textToSpeechPage">
        <h1><span class="title">SpeakEasy</span></h1>
        <h2>Text to Speech</h2>
        <label for="textInput">Enter the text to speak:</label>
        <input type="text" id="textInput" placeholder="Enter text...">
       
        <div id="languageSelection">
            <label for="english">
                <input type="radio" id="english" name="language" value="en-US" checked>
                English
            </label>
            <label for="tamil">
                <input type="radio" id="tamil" name="language" value="ta-IN">
                Tamil
            </label>
        </div>

        <button id="speakButton">Speak</button>
        <button id="pauseResumeButton">Pause</button>
        <br>
        <label for="rateInput">Speech Rate:</label>
        <input type="range" id="rateInput" min="0.5" max="2" step="0.1" value="1">
        <br>
        <label for="pitchInput">Speech Pitch:</label>
        <input type="range" id="pitchInput" min="0.5" max="2" step="0.1" value="1">
        <br>
        <button id="clearHistoryButton">Clear History</button>
       
        <div class="history">
            <h2>Speech History</h2>
            <div id="historyList"></div>
        </div>
        <button onclick="showPage('landingPage')">Back to Home</button>
    </div>

    <script>
        function showPage(pageId) {
            // Hide all pages
            document.querySelectorAll('.page').forEach(function(page) {
                page.style.display = 'none';
            });
            // Show the selected page
            document.getElementById(pageId).style.display = 'block';
        }

        document.addEventListener('DOMContentLoaded', function() {
            showPage('landingPage'); // Show the landing page initially

            var speechSynthesis = window.speechSynthesis;
            var speechUtterance = new SpeechSynthesisUtterance();
            var speakButton = document.getElementById('speakButton');
            var pauseResumeButton = document.getElementById('pauseResumeButton');
            var textInput = document.getElementById('textInput');
            var rateInput = document.getElementById('rateInput');
            var pitchInput = document.getElementById('pitchInput');
            var historyList = document.getElementById('historyList');

            rateInput.addEventListener('input', function() {
                speechUtterance.rate = parseFloat(rateInput.value);
            });

            pitchInput.addEventListener('input', function() {
                speechUtterance.pitch = parseFloat(pitchInput.value);
            });

            speakButton.addEventListener('click', function() {
                var text = textInput.value.trim();
                var lang = document.querySelector('input[name="language"]:checked').value;
                if (text !== '') {
                    textToSpeech(text, lang);
                } else {
                    textInput.style.borderColor = 'red';
                    setTimeout(function() {
                        textInput.style.borderColor = '#007bff';
                    }, 1000);
                }
            });

            pauseResumeButton.addEventListener('click', function() {
                if (speechSynthesis.paused) {
                    speechSynthesis.resume();
                    pauseResumeButton.textContent = 'Pause';
                } else {
                    speechSynthesis.pause();
                    pauseResumeButton.textContent = 'Resume';
                }
            });

            var clearHistoryButton = document.getElementById('clearHistoryButton');
            clearHistoryButton.addEventListener('click', function() {
                localStorage.removeItem('speechHistory');
                renderHistory();
                alert('Speech history cleared!');
            });

            function saveSpeechToHistory(text, lang) {
                var history = JSON.parse(localStorage.getItem('speechHistory')) || [];
                history.unshift({ text: text, lang: lang });
                localStorage.setItem('speechHistory', JSON.stringify(history));
                renderHistory();
            }

            function loadSpeechHistory() {
                return JSON.parse(localStorage.getItem('speechHistory')) || [];
            }

            function renderHistory() {
                historyList.innerHTML = '';
                var history = loadSpeechHistory();
                history.forEach(function(item) {
                    var historyItem = document.createElement('div');
                    historyItem.classList.add('history-item');
                    historyItem.textContent = item.text + ' (' + item.lang + ')';
                    historyList.appendChild(historyItem);
                });
            }

            renderHistory();

            function textToSpeech(text, lang) {
                var speechUtterance = new SpeechSynthesisUtterance(text);
                speechUtterance.lang = lang;
                speakButton.disabled = true;
                speakButton.textContent = 'Speaking...';
                speakButton.classList.add('speaking');

                speechSynthesis.speak(speechUtterance);

                speechUtterance.onend = function(event) {
                    speakButton.disabled = false;
                    speakButton.textContent = 'Speak';
                    speakButton.classList.remove('speaking');
                    textInput.value = '';
                    pauseResumeButton.textContent = 'Pause';
                    saveSpeechToHistory(text, lang);
                    alert('Speech synthesis complete!');
                };
            }

            // Speech to Text functionality
            var startRecordingButton = document.getElementById('startRecordingButton');
            var stopRecordingButton = document.getElementById('stopRecordingButton');
            var transcript = document.getElementById('transcript');
            var clearTranscriptButton = document.getElementById('clearTranscriptButton');
            var recognition;

            if (window.SpeechRecognition || window.webkitSpeechRecognition) {
                recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
                recognition.continuous = true;
                recognition.interimResults = true;

                recognition.onresult = function(event) {
                    var transcriptText = '';
                    for (var i = event.resultIndex; i < event.results.length; i++) {
                        transcriptText += event.results[i][0].transcript;
                    }
                    transcript.value = transcriptText;
                };

                recognition.onerror = function(event) {
                    console.error('Speech recognition error:', event.error);
                };
            } else {
                alert('Speech Recognition API is not supported in this browser.');
            }

            startRecordingButton.addEventListener('click', function() {
                if (recognition) {
                    recognition.start();
                    startRecordingButton.disabled = true;
                    stopRecordingButton.disabled = false;
                }
            });

            stopRecordingButton.addEventListener('click', function() {
                if (recognition) {
                    recognition.stop();
                    startRecordingButton.disabled = false;
                    stopRecordingButton.disabled = true;
                }
            });

            // Clear transcript button functionality
            clearTranscriptButton.addEventListener('click', function() {
                transcript.value = ''; // Clear the textarea
            });
        });
    </script>
</body>
</html>
