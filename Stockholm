var stockholm_geometry = KTH_Stockholm.geometry(); 
Map.addLayer(KTH_Stockholm,{'min':0,'max':1},'KTH_Stockholm',false);
///////////////////////////////////////////////////////////
var clip_func = function (img) { 
  return img.clip(stockholm_geometry) 
  .set('system:time_start', img.get('system:time_start'));
}; 
 
////////////////////////////////////////////////////
var weight_mean_func = function (img) {
  var days = ee.Number(((img.date()).difference(start_date, 'day')).uint16());
  var x = (days.float()).divide(366).multiply(4).subtract(2);
  var coef = ee.Number(1).divide(((x.multiply(-1)).exp()).add(1));
  return img.multiply(coef)
  .set('system:time_start', img.get('system:time_start'));
}; 
////////////////////////////////////////////////////

var date1 = '2016-01-01'; 
var date2 = '2017-01-01';

  var start_date = ee.Date(date1);
  var stop_date = ee.Date(date2);  
  
  var sentinel1_ASC= ee.ImageCollection('COPERNICUS/S1_GRD')
            .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
            .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
            .filter(ee.Filter.eq('relativeOrbitNumber_start', 102))
            .filterDate(start_date, stop_date)
            .filterBounds(stockholm_geometry)
            .sort("system:time_start");
 //  print(sentinel1_ASC) ;

   var weighted_collection_ASC = sentinel1_ASC.map(weight_mean_func);
//   Map.addLayer(ee.Image(weighted_collection_ASC.toList(500).get(25)).divide(ee.Image(sentinel1_ASC.toList(500).get(25))));
   
//Map.addLayer(ee.Image(sentinel1_ASC.toList(500).get(0)))
 //   Map.addLayer(weighted_collection_ASC.mean(), {},'list_ASC', false);
 weighted_collection_ASC = weighted_collection_ASC.map(clip_func);
////////////////////////////////////////////////////////////////////////////

  var sentinel1_DESC = ee.ImageCollection('COPERNICUS/S1_GRD')
            .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
            .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
            .filter(ee.Filter.eq('relativeOrbitNumber_start', 22))
            .filterDate(start_date, stop_date)
            .filterBounds(stockholm_geometry)
            .sort("system:time_start");
//   print(sentinel1_DESC) ;

  var weighted_collection_DESC = sentinel1_DESC.map(weight_mean_func);
 //    Map.addLayer(weighted_collection_DESC.mean(), {},'list_DESC', false);
 weighted_collection_DESC = weighted_collection_DESC.map(clip_func);
/////////////////////////////////////////////
/////////////////////////////

var getNewBandNames = function(prefix) {
  var seq = ee.List.sequence(1, bandNames.length());
  return seq.map(function(b) {
    return ee.String(prefix).cat(ee.Number(b).int());
  });
};

var getPrincipalComponents = function(centered, scale, region) {
  // Collapse the bands of the image into a 1D array per pixel.
  var arrays = centered.toArray();

  // Compute the covariance of the bands within the region.
  var covar = arrays.reduceRegion({
    reducer: ee.Reducer.centeredCovariance(),
    geometry: region,
    scale: scale,
    maxPixels: 1e9
  });

  // Get the 'array' covariance result and cast to an array.
  // This represents the band-to-band covariance within the region.
  var covarArray = ee.Array(covar.get('array'));

  // Perform an eigen analysis and slice apart the values and vectors.
  var eigens = covarArray.eigen();

  // This is a P-length vector of Eigenvalues.
  var eigenValues = eigens.slice(1, 0, 1);
  // This is a PxP matrix with eigenvectors in rows.
  var eigenVectors = eigens.slice(1, 1);

  // Convert the array image to 2D arrays for matrix computations.
  var arrayImage = arrays.toArray(1);

  // Left multiply the image array by the matrix of eigenvectors.
  var principalComponents = ee.Image(eigenVectors).matrixMultiply(arrayImage);

  // Turn the square roots of the Eigenvalues into a P-band image.
  var sdImage = ee.Image(eigenValues.sqrt())
    .arrayProject([0]).arrayFlatten([getNewBandNames('sd')]);

  // Turn the PCs into a P-band image, normalized by SD.
  return principalComponents
    // Throw out an an unneeded dimension, [[]] -> [].
    .arrayProject([0])
    // Make the one band array image a multi-band image, [] -> image.
    .arrayFlatten([getNewBandNames('pc')])
    // Normalize the PCs by their SDs.
    .divide(sdImage);
};
///////////////////////////////////////////////
//////////////////////////////////////////////////////////////////////////
 var terrain = ee.call('Terrain', ee.Image('USGS/SRTMGL1_003')).clip(stockholm_geometry);
var kernel_mountain =  ee.Kernel.circle({radius: 10});

// var terrain_normal = normalizationImage(terrain);
// print(terrain_normal);
var elevation_stdDev = terrain.select(['elevation'])
.reduceRegion(ee.Reducer.stdDev(), stockholm_geometry, 30, null, null, false, 1e13);
var elevation_stdDev_num = ee.Number(elevation_stdDev.get('elevation'));
var elevation_mean = terrain.select(['elevation'])
.reduceRegion(ee.Reducer.mean(), stockholm_geometry, 30, null, null, false, 1e13);
var elevation_mean_num = ee.Number(elevation_mean.get('elevation'));
//print('elevation_stdDev_num',elevation_stdDev_num);
//print('elevation_mean_num',elevation_mean_num);

var elevation_temp = terrain.select(['elevation'])
    .lt(elevation_mean_num);
      //    .focal_max({kernel: kernel_mountain, iterations: 2});
//Map.addLayer(elevation_temp,{},'elevation_temp',false);         

var slope_temp = terrain.select('slope').clip(stockholm_geometry).lt(10);
         //     .focal_max({kernel: kernel_mountain, iterations: 3})
            // .focal_min({kernel: kernel_mountain, iterations: 3});
// Map.addLayer(slope_temp,{},'slope_temp',false);   
 
 var LeLs = elevation_temp.and(slope_temp);
// Map.addLayer(LeLs,{},'LeLs',false);
 var LeHs = elevation_temp.and(slope_temp.not());
// Map.addLayer(LeHs,{},'LeHs',false);
 var HeLs = elevation_temp.not().and(slope_temp);
// Map.addLayer(HeLs,{},'HeLs',false);
 var HeHs = elevation_temp.not().and(slope_temp.not());
// Map.addLayer(HeHs,{},'HeHs',false);

///////////////////////////////////////
/////////// Weighted mean value of Night Light  ////////////////////////////

var night_light = night_collection
 .filterDate(start_date, stop_date)
 .select('avg_rad')
 .map(weight_mean_func);

  var night_light_mean = night_light
 .mean()
 .clip(stockholm_geometry);
 
 var min_max_light = night_light_mean.reduceRegion(ee.Reducer.minMax(), stockholm_geometry, 500, null,null,false,1e13);
  var num_min_light = ee.Number(min_max_light.get('avg_rad_min'));
  var num_max_light = ee.Number(min_max_light.get('avg_rad_max'));

 var mean_light = night_light_mean.reduceRegion(ee.Reducer.mean(), stockholm_geometry, 500, null,null,false,1e13);
  mean_light = ee.Number(mean_light.get('avg_rad'));
//  print('mean_light',mean_light);
  var stdDev_light = night_light_mean.reduceRegion(ee.Reducer.stdDev(), stockholm_geometry, 500, null,null,false,1e13);
  stdDev_light = ee.Number(stdDev_light.get('avg_rad'));
//  print('stdDev_light',stdDev_light);
  
  var num_min_light2 = mean_light.subtract(stdDev_light.multiply(3));
  var num_max_light2 =  mean_light.add(stdDev_light.multiply(3));
  
  num_min_light = num_min_light.max(num_min_light2);
  num_max_light = num_max_light.min(num_max_light2);
  
  
 var night_light_normal = (night_light_mean.subtract(num_min_light)).divide(num_max_light.subtract(num_min_light));
 night_light_normal = night_light_normal.clamp(0,1);
// Map.addLayer(night_light_normal,{},'night_light_normal', false) ;
