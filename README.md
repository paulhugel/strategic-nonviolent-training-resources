<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Strategic Nonviolent Training Resources Catalog</title>
  <link rel="stylesheet" href="styles.css" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
</head>
<body>
  <!-- ... other body content ... -->

  <!-- Mind Map Modal -->
  <div id="mindMapModal" class="modal">
    <div class="modal-header">
      <button id="tabStatic" class="tabBtn" onclick="showStatic()">Static SVG</button>
      <button id="tabDynamic" class="tabBtn" onclick="showDynamic()">Dynamic Graph</button>
      <button id="tabRadial360" class="tabBtn" onclick="showRadial360()">Radial 360</button>
      <div style="margin-left:auto; display:flex; gap:12px; align-items:center;">
        <label class="small">Spacing</label>
        <input id="spacingRange" type="range" min="60" max="260" value="140" oninput="updateSpacing(this.value)" />
        <label class="small">Radius</label>
        <input id="radiusRange" type="range" min="300" max="1400" value="820" oninput="updateRadius(this.value)" />
        <label class="small">Sector</label>
        <input id="sectorRange" type="range" min="12" max="40" value="24" oninput="updateSector(this.value)" />
      </div>
    </div>
    <div class="modal-body">
      <div id="staticView" class="mm-wrap">
        <!-- existing static SVG content -->
      </div>
      <div id="dynamicView" class="mm-wrap" style="display:none;">
        <!-- existing dynamic graph content -->
      </div>
      <div id="radial360View" class="mm-wrap" style="display:none;">
        <svg id="radial360Svg" width="100%" height="100%"></svg>
      </div>
    </div>
  </div>

  <style>
    /* existing modal and mindmap styles */

    .rad360-link { stroke:#334155; stroke-opacity:.7; fill:none; }
    .rad360-label { fill:#e5e7eb; font-size:13px; user-select:none; }
    .rad360-label.cat { font-weight:700; font-size:15px; }
    .rad360-node circle { fill:#0ea5e9; stroke:#0b132b; stroke-width:2; }
    .rad360-node.cat circle { fill:#1e40af; }
  </style>

  <script>
    // existing DATA and functions for static and dynamic views

    /* ==== Radial 360 (equal sectors, interactive) ==== */
    let radialSpacing = 140;
    let radialRadius = 820;
    let radialSectorDeg = 24;

    function buildEqualSectorData() {
      // group DATA by category -> [{name, items:[{name, href}...]}...]
      const cats = Array.from(new Set(DATA.map(d => d.category)));
      return cats.map(c => ({
        name: c,
        items: DATA.filter(d => d.category === c).map(r => ({ name: r.title, href: r.href }))
      }));
    }

    function renderRadial360() {
      const svg = d3.select('#radial360Svg');
      svg.selectAll('*').remove();

      const node = svg.node();
      const w = node.clientWidth || 1200;
      const h = node.clientHeight || 800;

      const cx = w / 2, cy = h / 2;

      const g = svg.append('g').attr('transform', `translate(${cx},${cy})`);

      const groups = buildEqualSectorData();
      const n = groups.length;
      if (!n) return;

      const full = 2 * Math.PI;
      const step = full / n;

      const innerR = Math.max(120, Math.min(w, h) * 0.22);
      const baseOuter = Math.min(radialRadius, Math.min(w, h)/2 - 32);

      // center label
      g.append('text')
        .attr('text-anchor','middle')
        .attr('fill','#93c5fd')
        .style('font-size','16px')
        .text('Nonviolent Training Resources');

      groups.forEach((grp, i) => {
        const catAng = i * step - Math.PI/2; // start at top
        const catX = Math.cos(catAng) * innerR;
        const catY = Math.sin(catAng) * innerR;

        // Category node
        const cat = g.append('g')
          .attr('class','rad360-node cat')
          .attr('transform', `translate(${catX},${catY})`);
        cat.append('circle').attr('r', 8);

        // Category label
        const labOff = 18;
        const anchor = (Math.cos(catAng) >= 0) ? 'start' : 'end';
        g.append('text')
          .attr('class','rad360-label cat')
          .attr('x', catX + Math.cos(catAng)*labOff)
          .attr('y', catY + Math.sin(catAng)*labOff)
          .attr('text-anchor', anchor)
          .attr('dy', '0.32em')
          .text(grp.name);

        // Resources spread within fixed sector around the category angle
        const items = grp.items.slice(); // copy
        const m = items.length;
        const sectorRad = (radialSectorDeg * Math.PI / 180) / 2; // +/- around center angle
        const startAng = catAng - sectorRad;
        const endAng = catAng + sectorRad;

        // compute how many we can put per ring based on spacing
        const perRing = Math.max(1, Math.floor((radialSectorDeg * Math.PI/180 * baseOuter) / radialSpacing));
        const rings = [];
        let idx = 0;
        while (idx < m) {
          const next = items.slice(idx, idx + perRing);
          rings.push(next);
          idx += perRing;
        }

        rings.forEach((ringItems, ringIdx) => {
          const R = baseOuter + ringIdx * (radialSpacing);
          const count = ringItems.length;
          for (let j = 0; j < count; j++) {
            const t = (count === 1) ? 0.5 : j / (count - 1);
            const ang = startAng + t * (endAng - startAng);
            const x = Math.cos(ang) * R;
            const y = Math.sin(ang) * R;

            const item = ringItems[j];

            // Link from category to item (smooth radial curve)
            const link = d3.linkRadial()
              .angle(d => d.a)
              .radius(d => d.r);
            const pathD = link({
              source: { a: catAng, r: innerR },
              target: { a: ang, r: R }
            });
            g.append('path')
              .attr('class','rad360-link')
              .attr('d', pathD);

            // Node
            const nodeG = g.append('g')
              .attr('class','rad360-node')
              .attr('transform', `translate(${x},${y})`)
              .on('click', () => { if (item.href) window.open(item.href, '_blank', 'noopener'); });
            nodeG.append('circle').attr('r', 6);

            // Label
            const la = (Math.cos(ang) >= 0) ? 'start' : 'end';
            g.append('text')
              .attr('class','rad360-label')
              .attr('x', x + (Math.cos(ang) >= 0 ? 10 : -10))
              .attr('y', y + 4)
              .attr('text-anchor', la)
              .text(item.name);
          }
        });
      });
    }

    // Tab handlers and sliders
    function showRadial360() {
      document.getElementById('staticView').style.display = 'none';
      document.getElementById('dynamicView').style.display = 'none';
      document.getElementById('radial360View').style.display = 'block';
      document.getElementById('tabRadial360').classList.add('active');
      document.getElementById('tabStatic').classList.remove('active');
      document.getElementById('tabDynamic').classList.remove('active');
      renderRadial360();
    }

    function updateSpacing(v) { radialSpacing = Number(v); if (document.getElementById('radial360View').style.display !== 'none') renderRadial360(); }
    function updateRadius(v)  { radialRadius  = Number(v); if (document.getElementById('radial360View').style.display !== 'none') renderRadial360(); }
    function updateSector(v)  { radialSectorDeg = Number(v); if (document.getElementById('radial360View').style.display !== 'none') renderRadial360(); }

    // When ?mindmap=1, open mind map and default to Radial 360
    (function() {
      const params = new URLSearchParams(window.location.search);
      if (params.get('mindmap') === '1') {
        setTimeout(function() {
          openMindMap();
          if (typeof showRadial360 === 'function') { showRadial360(); }
        }, 120);
      }
    })();
  </script>
</body>
</html>