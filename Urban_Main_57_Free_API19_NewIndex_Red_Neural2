 var CityCenter = /* color: #d63000 */ee.Geometry.Point([12.50825034085921, 41.869003889805526]);
 Map.centerObject(CityCenter,12);
// var CRSinfo = 'EPSG:32632';
 var CRSinfo = 'EPSG:4326';
////////////////////////////////////////////////////////////////////////
     
var minLabel = ui.Label({
  value: 'Area size (km^2):',
  style: {stretch: 'vertical'}
});


// Create panels to hold lon/lat values.
var lon = ui.Label();
var lat = ui.Label();
//panel.add(ui.minLabel([lon, lat], ui.Panel.Layout.flow('horizontal')));


var textbox = ui.Textbox({
  value: "500", 
  style: {width: '50px'}
});


var panel_bottom = ui.Panel({
  widgets: [minLabel, textbox],
  layout: ui.Panel.Layout.flow('horizontal')
});

var panel = ui.Panel();
panel.style().set('width', '250px');

var Dates = {
  '2016-01-01 To 2017-01-01': ['2016-01-01','2017-01-01'],
  '2016-04-01 To 2017-04-01': ['2016-04-01','2017-04-01'],
  '2016-07-01 To 2017-07-01': ['2016-07-01','2017-07-01'],
  '2016-10-01 To 2017-10-01': ['2016-10-01','2017-10-01'],
  '2017-01-01 To 2018-01-01': ['2017-01-01','2018-01-01'], 
  '2017-04-01 To 2018-04-01': ['2017-04-01','2018-04-01'], 
  '2017-07-01 To 2018-07-01': ['2017-07-01','2018-07-01'],
  '2017-10-01 To 2018-10-01': ['2017-10-01','2018-10-01'],
  '2018-01-01 To 2019-01-01': ['2018-01-01','2019-01-01'],
  '2018-04-01 To 2019-04-01': ['2018-04-01','2019-04-01'],
  '2018-07-01 To 2019-07-01': ['2018-07-01','2019-07-01'],
  '2018-10-01 To 2019-10-01': ['2018-10-01','2019-10-01'],
  '2019-01-01 To 2020-01-01': ['2019-01-01','2020-01-01'],
};

var start_date = ee.Date('2018-01-01');
var stop_date = ee.Date('2019-01-01');  
var Date_index1 = '2018-01-01';
var Date_index2 = '2019-01-01';
  
var selectDate = ui.Select({
  style: {width: '200px'},
  placeholder: 'Select the desired date...',
  items: Object.keys(Dates),
  onChange: function(key) {
   // Map.layers().reset();
     start_date = ee.Date(Dates[key][0]);
     stop_date = ee.Date(Dates[key][1]);
     Date_index1 = Dates[key][0];
     Date_index2 = Dates[key][1];
  }
});


 var Collection_city = ee.ImageCollection([]);
 

var places = {
  Beijing: [116.4386576304687,39.905744731816604,'Beijing','hand',0,0],
  Mexico: [-99.12533994426906,19.42810785771519,'Mexico','hand',0,0],
  Milan: [9.184083720947228,45.47812135157273,'Milan','hand',0,0],
  NewYork: [-74.11529806696655,40.6638355029954,'NewYork','hand',0,0],
  Rio: [-43.382435929375674,-22.80702839990601,'Rio','hand',0,0],
  Stockholm: [18.022221262206926,59.33443740876347,'Stockholm','hand',0,0],
  Others_Slow:[12.50825034085921, 41.869003889805526, 'Temp_App','crosshair',1,1]
};

var key1 = 0;
var select = ui.Select({
  style: {width: '200px'},
  placeholder: 'Select a region...',
  items: Object.keys(places),
  onChange: function(key) {
    Map.layers().reset();
    Map.setOptions("SATELLITE");
    Map.setCenter(places[key][0], places[key][1], 12);
    Map.style().set('cursor', places[key][3]);
    key1 = key;
  }
});

var Command = ui.Label({
  style: {stretch: 'vertical'}
});

var button = ui.Button({
  label: 'Find Urban Area Map',
  onClick: function() {
    Map.layers().reset();
    Map.setOptions("SATELLITE");
    Collection_city = 
    imageCollection
     .filterMetadata('system:index','contains',places[key1][2])
     .filterMetadata('system:index','contains', Date_index1)
     .filterMetadata('system:index','contains', Date_index2) ;
     var image_temp = ee.Image(Collection_city.toList(1).get(0)).eq(1);
     Map.addLayer(image_temp.updateMask(image_temp),{'min':0, 'max':1, 'palette':['ff0000']}, places[key1][2], true );
    var img_GHSL = (imageGHSL.select('built').gt(2)).clip(image_temp.geometry());
    Map.addLayer(img_GHSL.updateMask(img_GHSL),{min:0, max: 1, 'palette':['ff0000']},'GHSL JRC',false);
         
     
     var CommText = ee.String(ee.Algorithms.If(places[key1][5], 
     ee.String('Set the Area size and Click On The Map to Find the Urban Area'),
     ee.String('')));
   //  Command.setValue(CommText.getInfo());
       // Request the value from the server.
  CommText.evaluate(function(result) {
    // When the server returns the value, show it.
    panel.widgets().set(4, ui.Label({
      value: result,
    }));
  });
  }
});

panel.add(selectDate);
panel.add(select);
panel.add(button);
panel.add(panel_bottom);
panel.add(Command);

//panel.add(button);

// Register a callback on the default map to be invoked when the map is clicked.
Map.onClick(function(coords) {
  // Update the lon/lat panel with values from the click event.
  lon.setValue('lon: ' + coords.lon.toFixed(2)),
  lat.setValue('lat: ' + coords.lat.toFixed(2));

  // Add a red dot for the point clicked on.
  var point = ee.Geometry.Point(coords.lon, coords.lat);
  var dot = ui.Map.Layer(point, {color: 'FF0000'});
  
 
  Map.setOptions("SATELLITE");
  var layers = Map.layers();
  var layer = layers.get(1);
  layers.remove(layer);
  
  var buffer_size = Number(textbox.getValue());
  buffer_size = (Math.sqrt(buffer_size)/(0.002));
   var Buffered = point.buffer(buffer_size).bounds();

   var free_geometry = ee.FeatureCollection([ee.Feature(Buffered,{label: 'City', type: "City Center"})]);

var clip_func = function (img) {
  return img.clip(free_geometry)
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
///////////////////////////////////////////////
var NDVI_func = function (img) {
  var NDVI_image = (img.select('B8').subtract(img.select('B4')))
                .divide(img.select('B8').add(img.select('B4')));
  return NDVI_image
  .set('system:time_start', img.get('system:time_start'));
};

var NDWI_func = function (img) {
  var NDWI_image = (img.select('B3').subtract(img.select('B8')))
                .divide(img.select('B3').add(img.select('B8')));
  return NDWI_image
  .set('system:time_start', img.get('system:time_start'));
};

var NDSI_func = function (img) {
  var NDSI_image = (img.select('B3').subtract(img.select('B11')))
                .divide(img.select('B3').add(img.select('B11')));
  return NDSI_image
  .set('system:time_start', img.get('system:time_start'));
};

var CLOUD_func = function (img) {
  var mask_cloud = (img.select('QA60').neq(0).unmask()).not();
  return img.updateMask(mask_cloud)
  .set('system:time_start', img.get('system:time_start'));
};

///////////////////////////////////////////////
  var sentinel1_ASC = ee.ImageCollection('COPERNICUS/S1_GRD')
            .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
            .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
        //    .filter(ee.Filter.eq('relativeOrbitNumber_start', 142))
            .filterDate(start_date, stop_date)
            .filterBounds(free_geometry)
            .sort("system:time_start");
  var Size_ASC = (sentinel1_ASC.toList(500)).size().gte(5); 

////////////////////////////////////////////////////////////////////////////

  var sentinel1_DESC = ee.ImageCollection('COPERNICUS/S1_GRD')
            .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
            .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
       //     .filter(ee.Filter.eq('relativeOrbitNumber_start', 47))
            .filterDate(start_date, stop_date)
            .filterBounds(free_geometry)
            .sort("system:time_start");
  var Size_DESC = (sentinel1_DESC.toList(500)).size().gte(5);   
  /*
    ppp = sentinel1_DESC
        .aggregate_histogram('relativeOrbitNumber_start');
        //print('ppp',ppp);
    orbit_val = (ee.Number.parse(ee.Dictionary(ppp).keys().get(0)));
   sentinel1_DESC = sentinel1_DESC
        .filter(ee.Filter.eq('relativeOrbitNumber_start', orbit_val))  
*/
//////////////////////////////////////////////////////////////////////////////
  sentinel1_ASC = ee.ImageCollection(ee.Algorithms.If(Size_ASC, sentinel1_ASC, sentinel1_DESC));
//  print(sentinel1_ASC);
  sentinel1_DESC = ee.ImageCollection(ee.Algorithms.If(Size_DESC, sentinel1_DESC, sentinel1_ASC));
///////////////////////////////////////////////////////////////////////////////

var  sentinel1_ASC_Winter = sentinel1_ASC.filterDate(start_date, start_date.advance(3, 'month')).map(clip_func).median();
var  sentinel1_ASC_Spring = sentinel1_ASC.filterDate(start_date.advance(3, 'month'), start_date.advance(6, 'month')).map(clip_func).median();
var  sentinel1_ASC_Summer = sentinel1_ASC.filterDate(start_date.advance(6, 'month'), start_date.advance(9, 'month')).map(clip_func).median();
var  sentinel1_ASC_Fall = sentinel1_ASC.filterDate(start_date.advance(9, 'month'), stop_date).map(clip_func).median();

var  sentinel1_DESC_Winter = sentinel1_DESC.filterDate(start_date, start_date.advance(3, 'month')).map(clip_func).median();
var  sentinel1_DESC_Spring = sentinel1_DESC.filterDate(start_date.advance(3, 'month'), start_date.advance(6, 'month')).map(clip_func).median();
var  sentinel1_DESC_Summer = sentinel1_DESC.filterDate(start_date.advance(6, 'month'), start_date.advance(9, 'month')).map(clip_func).median();
var  sentinel1_DESC_Fall = sentinel1_DESC.filterDate(start_date.advance(9, 'month'), stop_date).map(clip_func).median();

var S1_ASC_Season_VV = sentinel1_ASC_Winter.select('VV')
        .addBands(sentinel1_ASC_Spring.select('VV'))
        .addBands(sentinel1_ASC_Summer.select('VV'))
        .addBands(sentinel1_ASC_Fall.select('VV'))
        .addBands(sentinel1_DESC_Winter.select('VV'))
        .addBands(sentinel1_DESC_Spring.select('VV'))
        .addBands(sentinel1_DESC_Summer.select('VV'))
        .addBands(sentinel1_DESC_Fall.select('VV'));
        
// Map.addLayer(S1_ASC_Season_VV,{},'S1_ASC_Season_VV',false);       
  
var S1_ASC_Season_VH = sentinel1_ASC_Winter.select('VH')
        .addBands(sentinel1_ASC_Spring.select('VH'))
        .addBands(sentinel1_ASC_Summer.select('VH'))
        .addBands(sentinel1_ASC_Fall.select('VH'))
        .addBands(sentinel1_DESC_Winter.select('VH'))
        .addBands(sentinel1_DESC_Spring.select('VH'))
        .addBands(sentinel1_DESC_Summer.select('VH'))
        .addBands(sentinel1_DESC_Fall.select('VH'));
        
// Map.addLayer(S1_ASC_Season_VH,{},'S1_ASC_Season_VH',false);   

var mean_Season_ASC_VV = S1_ASC_Season_VV.reduce(ee.Reducer.mean())//.focal_mode({radius:1,units:'pixels'});
var stdDev_Season_ASC_VV = S1_ASC_Season_VV.reduce(ee.Reducer.stdDev())//.focal_mode({radius:1,units:'pixels'}).focal_max({radius:1,units:'pixels'});
var mean_Season_ASC_VH = S1_ASC_Season_VH.reduce(ee.Reducer.mean())//.focal_mode({radius:1,units:'pixels'});
var stdDev_Season_ASC_VH = S1_ASC_Season_VH.reduce(ee.Reducer.stdDev())//.focal_mode({radius:1,units:'pixels'}).focal_max({radius:1,units:'pixels'});

// Map.addLayer(mean_Season_ASC_VV,{},'mean_Season_ASC_VV',false);   
//  Map.addLayer(stdDev_Season_ASC_VV,{},'stdDev_Season_ASC_VV',false);   
//   Map.addLayer(mean_Season_ASC_VH,{},'mean_Season_ASC_VH',false);   
//    Map.addLayer(stdDev_Season_ASC_VH,{},'stdDev_Season_ASC_VH',false);  

//////////////////////////////////////////////////////////////////////////////////////////////////
var sentinel2 = ee.ImageCollection('COPERNICUS/S2')
          //  .filter(ee.Filter.eq('SENSING_ORBIT_NUMBER', 95))
            .filterMetadata("CLOUDY_PIXEL_PERCENTAGE","less_than",10)
            .filterDate(start_date, stop_date)
            .filterBounds(free_geometry)
            .sort('system:time_start');

//   print(sentinel2) ;
sentinel2 = sentinel2.map(clip_func).map(CLOUD_func);
var NDVI_image = sentinel2.map(NDVI_func).max().clip(free_geometry);

var Green_removal = NDVI_image.gt(0.55);
//Map.addLayer(Green_removal.updateMask(Green_removal),{},'Green_removal',false); 

var NDWI_image = sentinel2.map(NDWI_func).median().clip(free_geometry); 
//Map.addLayer(NDWI_image,{min:0, max:1},'NDWI_image',false);
var Water_removal = NDWI_image.gt(0.2);
//Map.addLayer(Water_removal.updateMask(Water_removal),{},'Water_removal',false); 

var NDSI_image = sentinel2.map(NDSI_func).median().clip(free_geometry); 
//Map.addLayer(NDSI_image,{min:0, max:1},'NDSI_image',false);
var Snow_removal = NDSI_image.gt(0.3);
//Map.addLayer(Snow_removal.updateMask(Snow_removal),{},'Snow_removal',false); 

//////////////////////////////////////////////////////////////////////////////////


var  sentinel2_Winter = sentinel2.filterDate(start_date, start_date.advance(3, 'month'));
var  sentinel2_Spring = sentinel2.filterDate(start_date.advance(3, 'month'), start_date.advance(6, 'month'));
var  sentinel2_Summer = sentinel2.filterDate(start_date.advance(6, 'month'), start_date.advance(9, 'month'));
var  sentinel2_Fall =   sentinel2.filterDate(start_date.advance(9, 'month'), stop_date);

var  NDVI_Winter = sentinel2_Winter.map(NDVI_func).max().clip(free_geometry);
var  NDVI_Spring = sentinel2_Spring.map(NDVI_func).max().clip(free_geometry);
var  NDVI_Summer = sentinel2_Summer.map(NDVI_func).max().clip(free_geometry);
var  NDVI_Fall =  sentinel2_Fall.map(NDVI_func).max().clip(free_geometry);

var NDVI_Season = NDVI_Winter
        .addBands(NDVI_Spring)
        .addBands(NDVI_Summer)
        .addBands(NDVI_Fall);
//Map.addLayer(NDVI_Season,{},'NDVI_Season',false);   

var mean_Season_NDVI = NDVI_Season.reduce(ee.Reducer.mean());
var stdDev_Season_NDVI = NDVI_Season.reduce(ee.Reducer.stdDev());

// Map.addLayer(mean_Season_NDVI,{},'mean_Season_NDVI',false);   
//  Map.addLayer(stdDev_Season_NDVI,{},'stdDev_Season_NDVI',false);  

var NDVI_Season_Index = mean_Season_NDVI.multiply(stdDev_Season_NDVI);
//  Map.addLayer(NDVI_Season_Index,{},'NDVI_Season_Index',false);
  
  
///////////////////////////////////////
/////////// Weighted mean value of Night Light  ////////////////////////////

var night_light = ee.ImageCollection("NOAA/VIIRS/DNB/MONTHLY_V1/VCMCFG")
 .filterDate(start_date, stop_date)
 .select('avg_rad');
// .map(weight_mean_func);

  var night_light_mean = night_light
 .mean()
 .clip(free_geometry);
 
 night_light_mean = night_light_mean.clamp(0,5).divide(5);
 
var temp1 = (mean_Season_ASC_VV).subtract(stdDev_Season_ASC_VV);
// Map.addLayer(temp1,{},'temp1',false);  
var temp2 = (mean_Season_ASC_VH).subtract(stdDev_Season_ASC_VH);
// Map.addLayer(temp2,{},'temp2',false);  


/////////////////////////////////////////////////////////////////////////////////////////////
 var Multiband = night_light_mean.rename('night').addBands(NDVI_image.rename('ndvi')).addBands(NDWI_image.rename('ndwi'))
 .addBands(NDSI_image.rename('ndsi')).addBands(NDVI_Season_Index.rename('ndviSeason'))
 .addBands(temp1.rename('VV')).addBands(temp2.rename('VH')) ;
 
 
 ///////////////// Input layer //////////////////////////////////////////////////////////////////////

var X1_offset =  (ee.List([ee.Number(0.011),ee.Number(-0.422045678),ee.Number(-0.715538681),ee.Number(-0.616700292),ee.Number(-0.091164306),ee.Number(-47.02613701),ee.Number(-47.72078251)]));



X1_offset = ee.Image.constant(X1_offset).clip(free_geometry).reproject({crs:CRSinfo, scale:10}) ;
//print(X1_offset)
//print(ee.Image.constant(X1_offset))
//Map.addLayer(ee.Image.constant(X1_offset))
var X1_gain =  (ee.List([ee.Number(2.02224469160768),ee.Number(1.40698901900365),ee.Number(1.31721045085467),ee.Number(1.25802114992831),ee.Number(6.22660625115153),ee.Number(0.034569908591366),ee.Number(0.0422917871551224)]));


X1_gain = ee.Image.constant(X1_gain).clip(free_geometry).reproject({crs:CRSinfo, scale:10}) ;

var X1_ymin = -1;


var Xp1 = ((Multiband.subtract(X1_offset)).multiply(X1_gain)).add(X1_ymin);
//Map.addLayer(Xp1,{},'Xp1',false);
//print('Xp1',Xp1);

//var array1D = ee.Array([1, 2, 3,4 , 5, 6, 7 ,8]);              // [1,2,3]
//print(array1D);
//var array2D = ee.Array.cat([array1D, array1D.multiply(2)], 1);     // [[1],[2],[3]]
//print(array2D)

/////////////////// Hidden Layer 1 ///////////////////////////////////////////////////////////////////////////////////////

var IW1_1 = ee.Array([
[0.16355524010539207458,2.4559657299253223606,0.75677535714763388697,-1.0927823247012142804,0.15367865277314840533,-0.044956651152942617156,0.67763250463563995396],
[0.40819534142600327753,0.86299606508835169372,-0.49294449268856010971,-0.89442705373964004334,-1.1242133395716897848,0.060637832727913024145,-1.1375106928037146403],
[0.41003903464175128768,-0.65891220605368427954,0.7375937511343372277,-0.85053058823075244899,0.58745190498182908723,-0.29822944110919785698,0.17943147754786012427],
[-1.7576528853293356125,-0.48617718578150004305,-1.4564706974982237764,0.34673648737935697239,0.61269080913004680955,-0.80357596426819499769,-1.4601572738788111128],
[0.89199431219798630543,-0.49981248793835120203,0.34181313768147281174,0.89798116487809798159,0.97313427056020018746,-1.1019181546586660492,-1.1441038987622924594],
[0.73727212798714913955,-0.86249943287127106561,-0.61296821688244240711,-0.4781213866477975194,1.0301741392441778888,1.8905236821326480978,1.0935015850347118427],
[1.8131215621109966207,-1.3628336246312275915,1.1312179219158002841,-4.7859820202997633842,-0.59587994274436129061,-0.51086242195466813332,-0.051126919167539128241],
[-2.0897898274574786548,-0.070409778419505880676,-0.024010890887607286293,0.09032975180148669625,-0.24459585891377347289,-0.2816420732363250079,0.39262966427641082801],
[2.0867460113877309702,1.5670679006357857155,0.41019797842832728119,-0.30751434942467992251,-0.73506657066219360797,-0.0049246002275860015912,0.36424226201592185825],
[-1.4523107149144924843,0.4678826234149929264,-1.2337544241497073738,1.2911745751146117556,-0.8041163700397528924,-0.94592764643432003524,1.2283409865025654018],
[-0.75634396892345423513,0.42823598971226006782,0.41471068541585304201,-0.48941408837838301649,-0.73209816441202368864,-0.46679159912771112095,2.4063177926836698539],
[-0.37019606749338290763,-2.2176410134925195194,-0.37068769711874316464,-0.90854475949715218785,0.99266253033984619414,0.91504641764825656036,-1.6141965710213392882],
[-2.8603011699715610305,3.5832149317942367794,-0.31270341744041046939,0.22882085123271878047,-0.3381381584467318957,-0.71586064360843271182,-1.167996798425642524],
[1.3386072365349004354,0.4661477421192672943,-0.72690246790717771841,3.5002191542132523594,0.20071558933927585722,0.5422226427674641247,1.1731805729401572069],
[-0.26191567034190249563,-0.0037499873343104618909,-0.21410093325971860101,0.31572379471334360845,0.12436581654101799832,-0.99322177739543315855,-0.78629846889016030698]
]);

 
// Make an Array Image, with a 1-D Array per pixel.
var arrayImage1D = Xp1.toArray();
// Make an Array Image with a 2-D Array per pixel, 6x1.
var arrayImage2D = arrayImage1D.toArray(1);
//print(arrayImage2D)
//Map.addLayer(arrayImage2D)
//ee.Image(coefficients)

//var temp = ee.Image(coefficients).matrixMultiply(arrayImage2D);
//print('temp',temp);
//Map.addLayer(temp)

// Do a matrix multiplication: 6x6 times 6x1.
var componentsImage = ee.Image(IW1_1)
  .matrixMultiply(arrayImage2D)
  // Get rid of the extra dimensions.
  .arrayProject([0])
  .arrayFlatten(
    [['01', '02', '03', '04', '05', '06', '07', '08','09','10','11','12','13','14','15']]);
//Map.addLayer(componentsImage,{},'componentsImage',false);
//print(componentsImage)

var b1_array =  (ee.List([ee.Number(-2.8709810184340760486),ee.Number(1.6463722707303551918),ee.Number(2.0457540098214761493),ee.Number(-2.2606292806947037022),
ee.Number(-1.523676297601622931),ee.Number(-0.34585009206996614184),ee.Number(0.56583273041967863115),ee.Number(-1.1470986212198166498),ee.Number(1.6800684287563509844),
ee.Number(-1.4353199485683343362),ee.Number(-2.1355158971748315899),ee.Number(-0.75052176575608686715),ee.Number(-3.1862596469447694858),ee.Number(2.5351984470670134719),
ee.Number(-2.5091931288438322767)]));


var b1 = ee.Image.constant(b1_array).clip(free_geometry).reproject({crs:CRSinfo, scale:10}) ;
//Map.addLayer(b1,{},'b1',false);

var n = componentsImage.add(b1);
//Map.addLayer(n,{},'n',false);

var nEXP = (n.multiply(-2)).exp();
var a1 = (nEXP.subtract(1)).divide(nEXP.add(1)).multiply(-1);
//Map.addLayer(a1,{},'a1',false);

/////////////////// Hidden Layer 2 ///////////////////////////////////////////////////////////////////////////////////////
var LW2_1 = ee.Array([
[0.81939790441223514517,1.2384825075277468009,0.83631781527160709011,-0.85032169400959911609,0.53545558854919350633,-0.26279805037494602393,-0.73556547596678178991,
-1.6706862404763791474,-1.6830614417141813721,0.72969092177115613129,0.84544593392833478074,0.24255405230329721289,0.70391404915286803767,-1.0567089330126675506,
-0.77554132801422948074],[-0.81239065627432860417,-1.1983049592571528574,-0.55375057781235548227,0.83548694433155668015,-0.58221345383979261623,0.25122179125758331564,
0.72880078377882850926,1.6423283480941928136,1.6846412409635971308,-0.67783884448750542084,-0.84717082877397709151,-0.23224018821797487444,-0.69919711627127711928,
1.0555357414792243542,1.0773404463227602701]
]);


// Make an Array Image, with a 1-D Array per pixel.
var arrayImage1D = a1.toArray();
// Make an Array Image with a 2-D Array per pixel, 6x1.
var arrayImage2D = arrayImage1D.toArray(1);

var componentsImage2 = ee.Image(LW2_1)
  .matrixMultiply(arrayImage2D)
  // Get rid of the extra dimensions.
  .arrayProject([0])
  .arrayFlatten(
    [['01', '02']]);
//Map.addLayer(componentsImage2,{},'componentsImage2',false);


var b2_array = (ee.List([ee.Number(1.0921058354290604786),ee.Number(-1.1112927642829775188)]));


var b2 = ee.Image.constant(b2_array).clip(free_geometry).reproject({crs:CRSinfo, scale:10}) ;
//Map.addLayer(b2,{},'b2',false);

var a2 = componentsImage2.add(b2);
//Map.addLayer(a2,{},'a2',false);
/////////////////////////////  Output Layer ////////////////////////////////////////////////////////////////////////////////////

var y1 =  (a2.add(1)).divide(2);
//Map.addLayer(y1,{},'y1',false);

/*
var sequence = ee.List.sequence(1, 46, 1);
var func_02 = function (sequence){
   var S_extrema  = ee.Image.constant(sequence);
  return S_extrema.clip(free_geometry);
};
var temp2 = ee.ImageCollection(sequence.map(func_02)).toBands();
Map.addLayer(temp2,{},'temp2',false);
*/

var y2 = (y1.select('02').subtract(y1.select('01'))).gt(0).and(Green_removal.not());
//Map.addLayer(y2,{},'y2',false);
Map.addLayer(y2.updateMask(y2),{min:0, max: 1, 'palette':['ff0000']},'Urban Area',true);



var img_GHSL = (imageGHSL.select('built').gt(2)).clip(free_geometry);
Map.addLayer(img_GHSL.updateMask(img_GHSL),{min:0, max: 1, 'palette':['ff0000']},'GHSL JRC',false);
//var Final_urban = classified.or(Urban_NDVI).and(Green_removal.not());
//Map.addLayer(Final_urban,{min:0, max: 1},'Final_urban',false);
//Map.addLayer(Final_urban.updateMask(Final_urban),{min:0, max: 1},'Final_urban',true);  
  

});


// Add the panel to the ui.root.
ui.root.insert(0, panel);
