touch .nojekyll
git add .nojekyll
git commit -m "Disable Jekyll"
git push
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>2026</title>

  <style>
    *, *::before, *::after {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    html, body {
      width: 100%;
      height: 100%;
      overflow: hidden;
    }

    body {
      background-color: #A8CCDF;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .year {
      position: fixed;
      top: 30px;
      left: 34px;
      font-family: Arial, sans-serif;
      font-size: 22px;
      font-weight: 700;
      letter-spacing: 4px;
      color: rgba(255, 255, 255, 0.60);
      user-select: none;
    }

    #flag {
      display: block;
    }
  </style>
</head>
<body>

  <div class="year">2026</div>
  <canvas id="flag" width="130" height="68"></canvas>

  <script>
    const canvas = document.getElementById('flag');
    const ctx = canvas.getContext('2d');

    const W  = canvas.width;    // 130
    const H  = canvas.height;   // 68
    const B  = 3;               // block size in pixels
    const GW = W / B;           // 40 grid columns
    const GH = Math.floor(H / B); // 21 grid rows

    const COLOR_RED   = '#B22234';
    const COLOR_WHITE = '#FFFFFF';
    const COLOR_BLUE  = '#3C3B6E';

    // Canton: 2/5 of width, 7/13 of height
    const CW = Math.round(GW * 2 / 5);       // 16 columns
    const CH = Math.round(GH * 7 / 13);      // ~11 rows

    // Star positions within canton (0-indexed grid coords)
    // 5-star rows / 4-star rows alternating pattern
    const ROWS_5 = [1, 3, 5, 7, 9];
    const COLS_5 = [1, 4, 7, 10, 13];
    const ROWS_4 = [2, 4, 6, 8];
    const COLS_4 = [2, 5, 8, 11];

    function isStar(gx, gy) {
      if (gx >= CW || gy >= CH) return false;
      if (ROWS_5.includes(gy) && COLS_5.includes(gx)) return true;
      if (ROWS_4.includes(gy) && COLS_4.includes(gx)) return true;
      return false;
    }

    function getFlagColor(gx, gy) {
      if (gx < CW && gy < CH) {
        return isStar(gx, gy) ? COLOR_WHITE : COLOR_BLUE;
      }
      const stripeIndex = Math.floor(gy * 13 / GH);
      return stripeIndex % 2 === 0 ? COLOR_RED : COLOR_WHITE;
    }

    // Pre-build color lookup grid
    const grid = [];
    for (let gy = 0; gy < GH; gy++) {
      grid[gy] = [];
      for (let gx = 0; gx < GW; gx++) {
        grid[gy][gx] = getFlagColor(gx, gy);
      }
    }

    let t = 0;

    function draw() {
      ctx.clearRect(0, 0, W, H);

      for (let gx = 0; gx < GW; gx++) {
        // Wave: amplitude increases left-to-right (staff is on left)
        const progress  = gx / (GW - 1);
        const amplitude = progress * 1.7;
        const rawOffset = Math.sin(gx * 0.42 - t) * amplitude;

        // Snap to whole-block steps for true pixel-art feel
        const snapOffset = Math.round(rawOffset);

        for (let gy = 0; gy < GH; gy++) {
          const destY = gy + snapOffset;
          if (destY < 0 || destY >= GH) continue; // clip out-of-bounds rows
          ctx.fillStyle = grid[gy][gx];
          ctx.fillRect(gx * B, destY * B, B, B);
        }
      }

      t += 0.038;
      requestAnimationFrame(draw);
    }

    draw();
  </script>

</body>
</html>
