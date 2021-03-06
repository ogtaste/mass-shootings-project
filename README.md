<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <title>Mass Shootings in DC Since 2014</title>
  <meta name='viewport' content='width=device-width, initial-scale=1' />
  <script src='https://api.mapbox.com/mapbox-gl-js/v2.4.1/mapbox-gl.js'></script>
  <link href='https://api.mapbox.com/mapbox-gl-js/v2.4.1/mapbox-gl.css' rel='stylesheet' />

  <div id='map'></div>
  <div id='console'>
    <h1>Mass Shootings in Washington, DC</h1>
    <h3>2015 - 2021</h3>
    <h3>__</h3>
    <br>
    <p>Analysis & Visualization: <a href='http://dcwitness.org'>D.C. Witness</a><br>Data Sources: <a
        href='http://gunviolencearchive.org'>Gun Violence Archive</a>, D.C. Witness, Metropolitan Police Department
      (MPD)</p>
    <!-- <p>Visualization: <a href='http://twitter.com/mark_csv'>@mark_csv/Twitter</a></p> -->
    <div class='session'>
      <h2>Total Casualties (Fatal & Non-Fatal)</h2>
      <div class='row colors'>
      </div>
      <div class='row labels'>
        <div class='label'>4</div>
        <div class='label'>5</div>
        <div class='label'>6</div>
        <div class='label'>7</div>
        <div class='label'>8</div>
        <div class='label'>9+</div>
      </div>
    </div>
    <div class='session' id='sliderbar'>
      <h2>Year: <label id='year_slider'>[--select a year--]</label></h2>
      <input id='slider' class='row' type='range' min='2015' max='2021' step='1' value='2015' />
    </div>
    <div class='story-panel'>
      <div><strong>DATE: </strong><span id='incdate'></span></div>
      <div><strong>LOCATION: </strong><span id='incloc'></span></div>
      <div><strong>NEIGHBORHOOD: </strong><span id='inchood'></span></div>
      <div><strong>STORY: </strong><span id='incstory'></span></div>
    </div>

  </div>

  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Montserrat';
    }

    .story-panel {
      font-family: 'Montserrat';
      margin-top: 20px;
      margin-bottom: 10px;
      font-size: 12px;
      color: #222;
      background-color: #fff;
    }

    #map {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
    }


    h1 {
      font-size: 20px;
      line-height: 20px;
    }

    h2 {
      font-size: 10px;
      line-height: 20px;
      margin-bottom: 0px;
    }

    h3 {
      font-size: 15px;
      line-height: 0px;
      margin-bottom: 10px;
    }

    p {
      font-size: 12px;
      line-height: 14px;
      margin-bottom: 10px
    }

    a {
      text-decoration: none;
      color: #2dc4b2;
    }

    #console {
      position: absolute;
      width: 240px;
      margin: 10px;
      padding: 10px 20px;
      background-color: white;
    }

    .session {
      margin-bottom: 20px;
    }

    .row {
      height: 12px;
      width: 100%;
    }

    .colors {
      background: linear-gradient(to right, #d2e3f6, #0047ab);
      margin-bottom: 5px;
    }

    .label {
      width: 15%;
      display: inline-block;
      text-align: center;
    }

    .mapboxgl-popup {
      max-width: 400px;
      font: 12px 'Montserrat', Arial, Helvetica, sans-serif;
    }
  </style>

</head>

</body>

<script>
  mapboxgl.accessToken = 'pk.eyJ1IjoibWFya3dpdG5lc3MiLCJhIjoiY2tzbm50cDFvMTJtbjJubW12NWVrb2lwciJ9.iKSgMa83sgfiJvme61j4eA';

  const bounds = [
      [-77.23038331298717, 38.721611408010936],
      [-76.8273597126992, 39.08915516084373]
  ];

  const map = new mapboxgl.Map({
    container: 'map', // container element id
    style: 'mapbox://styles/markwitness/cku34xtp60bh917qjf5o6qzi2',
    center: [-77.04, 38.907], // initial map center in [lon, lat]
    zoom: 8,
    maxBounds: bounds
  });

  let hoveredStateId = null;

  map.on('load', () => {

    map.addSource('mass_shootings', {
      'type': 'geojson',
      'data': 'https://raw.githubusercontent.com/ogtaste/mass-shootings-project/main/MS_P5_D5.geojson'
    });

    map.addLayer({

      id: 'ms_main',
      type: 'circle',
      source: 'mass_shootings',
      paint: {
        'circle-stroke-color': 'Black',
        'circle-stroke-width': 1,
        'circle-radius':
          [
            "interpolate",
            ["linear"],
            ["get", "total_casualties"],
            4,
            10,
            5,
            13,
            6,
            17,
            7,
            19,
            8,
            24,
            9,
            28,
            22,
            45
          ],
        'circle-color':
          [
            "interpolate",
            ["linear"],
            ["get", "total_casualties"],
            4,
            "#d2e3f6",
            5,
            "#96b6e3",
            6,
            "#7aa0d9",
            7,
            "#608acf",
            8,
            "#4673c4",
            9,
            "#2a5db8",
            22,
            "#004a71"
          ],
        'circle-opacity': 0.75
      }

    });

    // An absolute ace in the hole on the first try, big data hours #GRINDING

    map.addSource('dc-outline', {
      'type': 'geojson',
      'data': 'https://raw.githubusercontent.com/ogtaste/mass-shootings-project/main/Washington_DC_Boundary.json'
    });





    map.addLayer({
      'id': 'boundary',
      'type': 'line',
      'source': 'dc-outline',
      'paint': {
        'line-color': '#483f2e',
        'line-width': 3
      }
    })

    // I LOVVVEEEE variables, dude. Can't get enough of them.


    map.on('click', 'ms_main', function (e) {
      var coordinates = e.features[0].geometry.coordinates.slice();
      var location = e.features[0].properties.address;
      var date = e.features[0].properties.date;
      var killed = e.features[0].properties.deaths;
      var numkilled = parseInt(e.features[0].properties.killed);
      var victims = e.features[0].properties.shot_individuals;
      var hoods = e.features[0].properties.neighborhoods;
      var context = e.features[0].properties.context;
      var total_casualties = e.features[0].properties.total_casualties;
      var sus = e.features[0].properties.suspect;
      var weapon = e.features[0].properties.weapon;
      var shot_individuals = e.features[0].properties.shot_individuals;
      var weapon = e.features[0].properties.weapon;
      var case_status = e.features[0].properties.case_status;

      // Ensure that if the map is zoomed out such that multiple
      // copies of the feature are visible, the popup appears
      // over the copy being pointed to.
      while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
        coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
      }

      new mapboxgl.Popup()
        .setLngLat(coordinates)
        .setHTML("<b>" + date + "</b>" + "<br>" + location + " (" + hoods + ")" + "<br>" + "<br>" + "<b>" + "VICTIMS:" + "</b>" + " " + victims + "<br>" + "<b>" + "DEATHS:" + "</b>" + " " + killed + "<br>" + "<b>" + "SUSPECT:" + "</b>" + " " + sus + "<br>" + "<b>" + "WEAPON:" + "</b>" + " " + weapon)
        .addTo(map);
    });

    // Change the cursor to a pointer when the mouse is over the places layer.
    map.on('mouseenter', 'ms_main', function () {
      map.getCanvas().style.cursor = 'pointer';
    });

    // Change it back to a pointer when it leaves.


    document.getElementById('slider').addEventListener('input', ({
      target }) => {
      const year1 = parseInt(target.value);

      // Updating the map
      map.setFilter('ms_main', ['==', ['number', ['get', 'inc_year']], year1]);
      document.getElementById('year_slider').innerText = year1;
    });

    let incID = null;

    map.on('mousemove', 'ms_main', (event) => {

      map.getCanvas().style.cursor = 'pointer';
      // Set constants equal to the current feature's magnitude, location, and time
      const incLocation = event.features[0].properties.address;
      const incNeighborhood = event.features[0].properties.neighborhoods;
      const incDate = event.features[0].properties.date;
      const incStory = event.features[0].properties.story;

      // Check whether features exist
      if (event.features.length === 0) return;
      // Display the magnitude, location, and time in the sidebar
      locationDisplay.textContent = incLocation;
      hoodDisplay.textContent = incNeighborhood;
      dateDisplay.textContent = incDate;
      storyDisplay.textContent = incStory;

      // If incID for the hovered feature is not null,
      // use removeFeatureState to reset to the default behavior
      if (incID) {
        map.removeFeatureState({
          source: 'ms_main',
          id: incID
        });
      }

      incID = event.features[0].id;

      // When the mouse moves over the earthquakes-viz layer, update the
      // feature state for the feature under the mouse
      map.setFeatureState(
        {
          source: 'ms_main',
          id: incID
        },
        {
          hover: true
        }
      );
    });

    map.on('mouseleave', 'ms_main', () => {
      if (incID) {
        map.setFeatureState(
          {
            source: 'ms_main',
            id: incID
          },
          {
            hover: false
          }
        );
      }

      incID = null;
      // Remove the information from the previously hovered feature from the sidebar
      locationDisplay.textContent = '';
      hoodDisplay.textContent = '';
      dateDisplay.textContent = '';
      storyDisplay.textContent = '';
      // Reset the cursor style
      map.getCanvas().style.cursor = '';
    });




  });

  const dateDisplay = document.getElementById('incdate');
  const locationDisplay = document.getElementById('incloc');
  const hoodDisplay = document.getElementById('inchood');
  const storyDisplay = document.getElementById('incstory');



</script>





</html>
