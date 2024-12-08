# DIAKITE
#FORECAST
#(NO₂, CO et O₃) à partir des données Sentinel-5P et générer des séries temporelles pour analyser l'évolution de la pollution atmosphérique à Bamako.
// Définir la région d'étude : Bamako
var bamako = ee.Geometry.Rectangle([-8.11, 12.50, -7.85, 12.70]); // Coordonnées approximatives de Bamako

// Charger les collections pour NO2, CO et O3
var no2Collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_NO2')
  .filterDate('2024-01-01', '2024-11-20') // Ajuster la plage temporelle
  .filterBounds(bamako);

var coCollection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_CO')
  .filterDate('2024-01-01', '2024-11-20')
  .filterBounds(bamako);

var o3Collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_O3')
  .filterDate('2024-01-01', '2024-11-20')
  .filterBounds(bamako);

// Moyenne des concentrations pour chaque gaz
var no2 = no2Collection.select('tropospheric_NO2_column_number_density').mean().clip(bamako);
var co = coCollection.select('CO_column_number_density').mean().clip(bamako);
var o3 = o3Collection.select('O3_column_number_density').mean().clip(bamako);

// Définir des paramètres de visualisation
var visParamsNo2 = { min: 0.0, max: 0.0002, palette: ['blue', 'green', 'yellow', 'orange', 'red'] };
var visParamsCo = { min: 0.0, max: 0.05, palette: ['black', 'purple', 'blue', 'green', 'yellow'] };
var visParamsO3 = { min: 0.0, max: 0.2, palette: ['white', 'cyan', 'blue', 'purple', 'magenta'] };

// Ajouter les couches à la carte
Map.centerObject(bamako, 12);
Map.addLayer(no2, visParamsNo2, 'Moyenne NO2');
Map.addLayer(co, visParamsCo, 'Moyenne CO');
Map.addLayer(o3, visParamsO3, 'Moyenne O3');

// Générer une série temporelle pour NO2
var no2TimeSeries = no2Collection.map(function(image) {
  var meanValue = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: bamako,
    scale: 1000
  }).get('tropospheric_NO2_column_number_density');
  return ee.Feature(null, {
    'date': image.date().format('YYYY-MM-dd'),
    'NO2_mean': meanValue
  });
});

var coTimeSeries = coCollection.map(function(image) {
  var meanValue = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: bamako,
    scale: 1000
  }).get('CO_column_number_density');
  return ee.Feature(null, {
    'date': image.date().format('YYYY-MM-dd'),
    'CO_mean': meanValue
  });
});

var o3TimeSeries = o3Collection.map(function(image) {
  var meanValue = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: bamako,
    scale: 1000
  }).get('O3_column_number_density');
  return ee.Feature(null, {
    'date': image.date().format('YYYY-MM-dd'),
    'O3_mean': meanValue
  });
});

// Combiner les séries temporelles
var timeSeries = no2TimeSeries.merge(coTimeSeries).merge(o3TimeSeries);

// Afficher les séries temporelles sous forme de graphique
print('Séries temporelles NO2', ui.Chart.feature.byFeature(no2TimeSeries, 'date', 'NO2_mean')
  .setOptions({
    title: 'Évolution du NO2 à Bamako',
    hAxis: { title: 'Date' },
    vAxis: { title: 'Concentration (mol/m²)' },
    lineWidth: 2,
    pointSize: 4
  }));

print('Séries temporelles CO', ui.Chart.feature.byFeature(coTimeSeries, 'date', 'CO_mean')
  .setOptions({
    title: 'Évolution du CO à Bamako',
    hAxis: { title: 'Date' },
    vAxis: { title: 'Concentration (mol/m²)' },
    lineWidth: 2,
    pointSize: 4
  }));

print('Séries temporelles O3', ui.Chart.feature.byFeature(o3TimeSeries, 'date', 'O3_mean')
  .setOptions({
    title: 'Évolution du O3 à Bamako',
    hAxis: { title: 'Date' },
    vAxis: { title: 'Concentration (mol/m²)' },
    lineWidth: 2,
    pointSize: 4
  }));
