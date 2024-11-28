# video-to-audio
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Video to Audio Converter with Progress</title>
  <script src="https://cdn.jsdelivr.net/npm/@ffmpeg/ffmpeg@0.10.0/dist/ffmpeg.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #f8f8f8;
      padding: 20px;
    }
    h1 {
      color: #333;
    }
    input[type="file"] {
      margin: 20px 0;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
      margin: 10px;
    }
    button:disabled {
      background-color: #ccc;
    }
    #progressContainer {
      margin-top: 20px;
    }
    #progressText {
      font-size: 18px;
      color: #007bff;
    }
    #downloadLink {
      margin-top: 20px;
      display: none;
    }
  </style>
</head>
<body>
  <h1>Video to Audio Converter</h1>
  <input type="file" id="videoFile" accept="video/*">
  <br>
  <button id="convertButton" onclick="convertVideoToAudio()" disabled>Convert to Audio</button>
  <button id="cancelButton" onclick="cancelConversion()" disabled>Cancel</button>
  <div id="progressContainer">
    <span id="progressText"></span>
  </div>
  <br>
  <a id="downloadLink" download="output.mp3">Download Audio File</a>

  <script>
    const videoInput = document.getElementById('videoFile');
    const convertButton = document.getElementById('convertButton');
    const cancelButton = document.getElementById('cancelButton');
    const progressText = document.getElementById('progressText');
    const downloadLink = document.getElementById('downloadLink');

    let ffmpeg = null;
    let conversionCanceled = false;

    videoInput.addEventListener('change', () => {
      convertButton.disabled = !videoInput.files.length;
    });

    async function convertVideoToAudio() {
      const file = videoInput.files[0];
      if (!file) {
        alert('Please select a video file first.');
        return;
      }

      convertButton.textContent = 'Converting...';
      convertButton.disabled = true;
      cancelButton.disabled = false;
      progressText.textContent = '0%';

      ffmpeg = FFmpeg.createFFmpeg({
        log: true,
        progress: ({ ratio }) => {
          if (!conversionCanceled) {
            const percentage = Math.round(ratio * 100);
            progressText.textContent = `${percentage}%`;
          }
        }
      });

      await ffmpeg.load();
      ffmpeg.FS('writeFile', 'input.mp4', await fetchFile(file));

      try {
        await ffmpeg.run('-i', 'input.mp4', '-q:a', '0', '-map', 'a', 'output.mp3');

        const data = ffmpeg.FS('readFile', 'output.mp3');
        const audioBlob = new Blob([data.buffer], { type: 'audio/mp3' });
        const url = URL.createObjectURL(audioBlob);

        downloadLink.href = url;
        downloadLink.style.display = 'block';

        progressText.textContent = 'Conversion completed!';
      } catch (error) {
        if (conversionCanceled) {
          progressText.textContent = 'Conversion canceled.';
        } else {
          progressText.textContent = 'Error occurred during conversion.';
        }
      } finally {
        convertButton.textContent = 'Convert to Audio';
        convertButton.disabled = false;
        cancelButton.disabled = true;
        ffmpeg = null;
        conversionCanceled = false;
      }
    }

    function cancelConversion() {
      if (ffmpeg) {
        conversionCanceled = true;
        ffmpeg.exit();
        progressText.textContent = 'Cancelling...';
        cancelButton.disabled = true;
      }
    }
  </script>
</body>
</html>
