<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Workspace Canvas</title>
  <style>
    html, body {
      margin: 0;
      height: 100%;
    }
    #container {
      display: flex;
      flex-direction: row;
      width: 100%;
      height: 100vh;
    }
    #toolbar {
      display: flex;
      flex-direction: column;
      width: 200px;
      padding: 10px;
      box-sizing: border-box;
      background: #f0f0f0;
    }
    #toolbar button {
      margin-bottom: 10px;
    }
    #workspace {
      flex: 1;
      position: relative;
      background: #fff;
    }
    #workspace canvas {
      width: 100%;
      height: 100%;
      border: 1px solid #ccc;
      display: block;
    }
  </style>
</head>
<body>
  <div id="container">
    <div id="toolbar">
      <button id="rectBtn">Draw Rectangle</button>
      <button id="circleBtn">Draw Circle</button>
      <button id="clearBtn">Clear</button>
    </div>
    <div id="workspace">
      <canvas id="canvas"></canvas>
    </div>
  </div>
  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const workspace = document.getElementById('workspace');

    function resizeCanvas() {
      canvas.width = workspace.clientWidth;
      canvas.height = workspace.clientHeight;
    }

    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    document.getElementById('rectBtn').addEventListener('click', () => {
      ctx.fillStyle = '#3498db';
      ctx.fillRect(10, 10, 100, 60);
    });

    document.getElementById('circleBtn').addEventListener('click', () => {
      ctx.fillStyle = '#e74c3c';
      ctx.beginPath();
      ctx.arc(150, 80, 40, 0, Math.PI * 2);
      ctx.fill();
    });

    document.getElementById('clearBtn').addEventListener('click', () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    });
  </script>
</body>
</html>
