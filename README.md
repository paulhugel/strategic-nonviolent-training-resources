<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Strategic Nonviolent Training Resources - Interactive Catalog</title>

<!-- Stylesheets -->
<link rel="stylesheet" href="styles.css" />

<!-- D3.js - only one include -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.9.0/d3.min.js" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

</head>
<body>

<header>
  <h1>Strategic Nonviolent Training Resources - Interactive Catalog</h1>
  <nav id="toolbar">
    <button id="tabList" onclick="showList()">List View</button>
    <button id="tabMindMap" onclick="openMindMap()">Mind Map</button>
    <button id="tabRadial360" onclick="showRadial360()">Radial 360</button>
    <button id="tabDynamic" onclick="showDynamic()">Dynamic Graph</button>
  </nav>
</header>

<main>
  <div id="content"></div>
</main>

<script>
// Mind Map / Dynamic / Radial code

// Radial 360 layout function
function showRadial360() {
  // ... setup code ...

  // Example loop with corrected comparison operators
  for (let j = 0; j < count; j++) {
    // loop body
  }

  // Another similar loop with corrected comparison operators
  for (let j = 0; j < countThisRing; j++) {
    // loop body
  }

  // ... rest of the radial 360 layout code ...
}

// Other view functions
function showList() {
  // ...
}
function openMindMap() {
  // ...
}
function showDynamic() {
  // ...
}

// Auto-open logic updated to open Radial 360 view by default when ?mindmap=1 is present
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
