---
title: "Home"
---

<div class="hero">
  <canvas id="fem-mesh" width="360" height="280"></canvas>
  <div class="hero-text">
    <p class="hero-greeting">Welcome.</p>
    <p>Hi there! My name is Muhammed (you can simply call me Moe) and here you will find information about my background, the projects I have worked on, and thoughts I choose to share with the world.</p>
    <p>Feel free to explore.</p>
    <p class="credit">Theme inspired by <a href="https://jorgemartinez.space/posts/journal/a-new-website-theme/" target="_blank" rel="noopener">Jorge Martinez's Unbloated</a>.</p>
  </div>
</div>

<script>
(function () {
  var canvas = document.getElementById("fem-mesh");
  if (!canvas) return;
  var ctx = canvas.getContext("2d");
  var W = canvas.width, H = canvas.height;

  // Grid parameters
  var cols = 14, rows = 10;
  var padX = 30, padY = 30;
  var spacingX = (W - 2 * padX) / (cols - 1);
  var spacingY = (H - 2 * padY) / (rows - 1);

  // Build rest positions
  var restX = [], restY = [];
  for (var j = 0; j < rows; j++) {
    for (var i = 0; i < cols; i++) {
      restX.push(padX + i * spacingX);
      restY.push(padY + j * spacingY);
    }
  }

  // Interpolate between two colors based on t [0,1]
  function lerpColor(r1, g1, b1, r2, g2, b2, t) {
    return [
      Math.round(r1 + (r2 - r1) * t),
      Math.round(g1 + (g2 - g1) * t),
      Math.round(b1 + (b2 - b1) * t)
    ];
  }

  // Contour color: blue (low) -> cyan -> green -> yellow -> red (high)
  function stressColor(val) {
    // val in [0, 1]
    var v = Math.max(0, Math.min(1, val));
    var r, g, b;
    if (v < 0.25) {
      var c = lerpColor(0, 112, 192, 0, 200, 220, v / 0.25);
      r = c[0]; g = c[1]; b = c[2];
    } else if (v < 0.5) {
      var c = lerpColor(0, 200, 220, 0, 180, 60, (v - 0.25) / 0.25);
      r = c[0]; g = c[1]; b = c[2];
    } else if (v < 0.75) {
      var c = lerpColor(0, 180, 60, 230, 210, 0, (v - 0.5) / 0.25);
      r = c[0]; g = c[1]; b = c[2];
    } else {
      var c = lerpColor(230, 210, 0, 210, 50, 30, (v - 0.75) / 0.25);
      r = c[0]; g = c[1]; b = c[2];
    }
    return "rgba(" + r + "," + g + "," + b + ",0.35)";
  }

  function idx(i, j) { return j * cols + i; }

  var t = 0;
  function animate() {
    t += 0.018;
    ctx.clearRect(0, 0, W, H);

    // Compute deformed positions — superposition of two bending modes
    var dx = [], dy = [];
    for (var j = 0; j < rows; j++) {
      for (var i = 0; i < cols; i++) {
        var u = i / (cols - 1);
        var v = j / (rows - 1);
        // Mode 1: first bending mode
        var amp1 = 8 * Math.sin(t * 1.0);
        var m1 = amp1 * Math.sin(Math.PI * u) * Math.sin(Math.PI * v);
        // Mode 2: second mode shape
        var amp2 = 5 * Math.sin(t * 1.7 + 1.2);
        var m2 = amp2 * Math.sin(2 * Math.PI * u) * Math.sin(Math.PI * v);
        // Mode 3: torsion-like
        var amp3 = 4 * Math.sin(t * 0.8 + 2.5);
        var m3x = amp3 * Math.sin(Math.PI * u) * Math.cos(Math.PI * v);

        var k = idx(i, j);
        dx.push(restX[k] + m3x);
        dy.push(restY[k] + m1 + m2);
      }
    }

    // Draw filled quads with stress contour
    for (var j = 0; j < rows - 1; j++) {
      for (var i = 0; i < cols - 1; i++) {
        var a = idx(i, j), b = idx(i + 1, j);
        var c = idx(i + 1, j + 1), d = idx(i, j + 1);
        // "Stress" from average displacement magnitude
        var avgDx = (dx[a] - restX[a] + dx[b] - restX[b] + dx[c] - restX[c] + dx[d] - restX[d]) / 4;
        var avgDy = (dy[a] - restY[a] + dy[b] - restY[b] + dy[c] - restY[c] + dy[d] - restY[d]) / 4;
        var mag = Math.sqrt(avgDx * avgDx + avgDy * avgDy);
        var stress = Math.min(mag / 12, 1);

        ctx.beginPath();
        ctx.moveTo(dx[a], dy[a]);
        ctx.lineTo(dx[b], dy[b]);
        ctx.lineTo(dx[c], dy[c]);
        ctx.lineTo(dx[d], dy[d]);
        ctx.closePath();
        ctx.fillStyle = stressColor(stress);
        ctx.fill();
      }
    }

    // Draw mesh edges
    ctx.strokeStyle = "rgba(0,112,192,0.7)";
    ctx.lineWidth = 1;
    for (var j = 0; j < rows; j++) {
      for (var i = 0; i < cols; i++) {
        var k = idx(i, j);
        // Horizontal edge
        if (i < cols - 1) {
          var k2 = idx(i + 1, j);
          ctx.beginPath();
          ctx.moveTo(dx[k], dy[k]);
          ctx.lineTo(dx[k2], dy[k2]);
          ctx.stroke();
        }
        // Vertical edge
        if (j < rows - 1) {
          var k2 = idx(i, j + 1);
          ctx.beginPath();
          ctx.moveTo(dx[k], dy[k]);
          ctx.lineTo(dx[k2], dy[k2]);
          ctx.stroke();
        }
      }
    }

    // Draw nodes
    ctx.fillStyle = "rgba(0,112,192,0.9)";
    for (var n = 0; n < dx.length; n++) {
      ctx.beginPath();
      ctx.arc(dx[n], dy[n], 2, 0, 2 * Math.PI);
      ctx.fill();
    }

    requestAnimationFrame(animate);
  }
  animate();
})();
</script>
