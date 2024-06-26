# Call any Feature Service
The City of Seattle offers some of its data as a REST service. Here I will show you how to create a mashup and call data directly from an Esri Server. In this tutorial, I will be using Seattle Open Data Esri Feature Services but you are welcome to find your own feature service. There are plenty out there!

This example is based on this <a href= "http://esri.github.io/esri-leaflet/tutorials/working-with-feature-layers.html">Esri Custom popup tutorial.</a> and <a href="https://esri.github.io/esri-leaflet/examples/feature-layer-popups.html">this example</a>. This is also a <a href= "https://github.com/Esri/esri-leaflet-renderers">helpful resource from ESRI</a> as the renders update frequently. 


Seattle offers many stylized feature services that you may incorporate use. I use <a href= "https://gisrevprxy.seattle.gov/arcgis/rest/services/ext/WM_CityGISLayers/MapServer/33">Tree data found here </a>, but you can browse other <a href= "https://gisrevprxy.seattle.gov/arcgis/rest/services/ext/WM_CityGISLayers/MapServer">Seattle data here</a>. Before you commit to a dataset you can browse what is contains when you click ArcGIS Online MapViewer on the data page. 

##Let's get started
I first provide you basic with can see we have the basic HTML requirements, the title of the page is Calling Features with Leaflet, the map is stylized to fill the whole screen, the center of the map is set to Seattle and zoom level is appropriately framing Seattle.

Start with the following: 

```
<!doctype html>
<html lang="en">
<head>  
  <meta charset="utf-8">
  <title>Leaflet Map with a Feature Layer</title>  
  <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />

  <!-- Load Leaflet from CDN-->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.0.1/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.0.1/dist/leaflet.js"></script>

  <!-- Load Esri Leaflet from Content Delivery Network (CDN) -->
  <script src="https://unpkg.com/esri-leaflet@2.0.4/dist/esri-leaflet.js"></script>

  <style>
    html,
    body,
    #map {
      height: 100%;
      width: 100%;
      margin: 0;
      padding: 0;
    }
  </style>
</head>
<body>    
    <div id="map"></div>

    <script>
        var map = L.map('map', {
          center: [47.626, -122.337],
          zoom: 13
        });

        var esriStreets = L.esri.basemapLayer('Topographic').addTo(map);    
    </script>    
</body>
</html>
```

This will provide us with a Topographic basemap that fills the browser frame.

Now let's add some data to feature! You have looked around what Seattle has to offer. As I mentioned, I am using the heritage tree data, so under the map variable and before I close the script tag I will name the variable seattleHeriageTrees to call the feature layer and add to the map. I add tags to remind me what the code does. 

```
 //set the tree variable to call the server to then display or add to the map
        var seattleHeritageTrees = L.esri.featureLayer({url: 'https://gisrevprxy.seattle.gov/arcgis/rest/services/ext/WM_CityGISLayers/MapServer/33'}).addTo(map);
        
 ```

If we open the code now, you will see the default blue pin markers and nothing else. Often they have symbology attached and you will be able to see from the Esri REST page if the symbology has been set. We need to tell the browser to get that style. To do so we add the following js call just before you close to the head. 

 ```
  <!-- Load Esri Leaflet Renderers -->
  <!-- This will hook into Esri Leaflet to get renderer info when adding a feature layer -->
    <script src="https://unpkg.com/esri-leaflet-renderers@2.0.6"></script>
 ```
Now the blue dots should be replaced with trees! 
 
Next, we need to add the popup window to say something meaningful. We do not want a laundry list of all the attributes that Esri Online provides. We only want to share what the end users need to see. I can see all the attributes available, from these, I decide I only want to share the common and scientific names of the tree selected.

I added the following just under the add to map. Again, I added comments to remember what I am showing.
 ```
 //customize the popup, do you no present a laundry list of attributes, think about what the end user needs to know.
        
 seattleHeritageTrees.bindPopup(function (layer) {
    return L.Util.template('<p>{DESCRIPT}<br>Its scientific name is {SCINAME}.</p>', layer.feature.properties);
  });
 ```
Hooray! Now when I click a tree, meaningful information pops up. This is fun so I want to add more information. 

#Add another layer 

I want to see if these trees are close to heritage landmarks. I can see that there is a Landmarks layer. I will make a new variable called Seattle landmarks and add it to the map. I add this next bit just under the add to map. 

 ```
 //set the Seattle Landmarks variable call the server to then display or add to the map
        var seattleLandmarks = L.esri.featureLayer({url: 'https://gisrevprxy.seattle.gov/arcgis/rest/services/ext/WM_CityGISLayers/MapServer/55'}).addTo(map);
 ```
 
 I need popup information for this new layer too. I see what attributes are available and decide to share information about the address and how long this site has been recognized. I add the following under the feature properties.
 
  ```
  //popup information for landmarks
 

            seattleLandmarks.bindPopup(function(layer) {
            return L.Util.template('<h3>{NAME}</h3><hr /><p>This landmark is located at {ADDRESS}.', layer.feature.properties);
                
         });       
          
  ```


Now we should see trees and landmarks. Fun! Let's keep going!
          
##Add base map selector

I want to user to be able to change the basemap. 

Add the following into the style tag. Make sure you place it properly, after the } the close of the #map

```
#basemaps-wrapper {
    position: absolute;
    top: 10px;
    right: 10px;
    z-index: 400;
    background: white;
    padding: 10px;
  }
  #basemaps {
    margin-bottom: 5px;
  }
 ```
 
 Then we need to add the div to place the dropdown within the map frame. Place this new div just under the map div and before you start writing your java script. 
 
  ```
 <div id="basemaps-wrapper" class="leaflet-bar">
  <select name="basemaps" id="basemaps" onChange="changeBasemap(basemaps)">
    <option value="Topographic">Topographic</option>
    <option value="Streets">Streets</option>
    <option value="NationalGeographic">National Geographic</option>
    <option value="Oceans">Oceans</option>
    <option value="Gray">Gray</option>
    <option value="DarkGray">Dark Gray</option>
    <option value="Imagery">Imagery</option>
    <option value="ShadedRelief">Shaded Relief</option>
  </select>
</div>
 ```
 
 Now we have to 
 
 Replace this code that renders the topographic Basemap...
   ```
   var esriStreets = L.esri.basemapLayer('Topographic').addTo(map); 
   
   ```
   With the following code
    ```
   var var layer = L.esri.basemapLayer('Topographic').addTo(map);
  var layerLabels;

  function setBasemap(basemap) {
    if (layer) {
      map.removeLayer(layer);
    }

    layer = L.esri.basemapLayer(basemap);

    map.addLayer(layer);

    if (layerLabels) {
      map.removeLayer(layerLabels);
    }

    if (basemap === 'ShadedRelief'
     || basemap === 'Oceans'
     || basemap === 'Gray'
     || basemap === 'DarkGray'
     || basemap === 'Imagery'
     || basemap === 'Terrain'
   ) {
      layerLabels = L.esri.basemapLayer(basemap + 'Labels');
      map.addLayer(layerLabels);
    }
  }

  function changeBasemap(basemaps){
    var basemap = basemaps.value;
    setBasemap(basemap);
  }

var layer = L.esri.basemapLayer('Topographic').addTo(map);
 
   ```
Now it will look great! 

##That is all for now
Here you have learned to call data straight from an Esri server instead of having to download and host the data yourself. Awesome! There are so many different ways to call feature layers. <a href="http://esri.github.io/esri-leaflet/tutorials/introduction-to-layer-types.html">See more examples here.</a> Enjoy!
