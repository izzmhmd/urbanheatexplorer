//CREATED BY IZZUDDIN MUHAMMAD (UNIVERSITAS NEGERI SEMARANG)
//IG: @adinizdn, LinkedIN: https://www.linkedin.com/in/izzmhmd/

//INDONESIA ADMINISTRAT AREA (GADM)
var ADMProv = ee.FeatureCollection('projects/ee-izzmhmd53/assets/Provinsi_IDN')
var ADMKab = ee.FeatureCollection('projects/ee-izzmhmd53/assets/Kabupaten_IDN')
var ADMKEC = ee.FeatureCollection('projects/ee-izzmhmd53/assets/Kecamatan_IDN')
//FAO ADM DATA
var FAO0 = ee.FeatureCollection('FAO/GAUL/2015/level0');
var FAO1 = ee.FeatureCollection('FAO/GAUL/2015/level1');

//VISUALIZATIONS
var landSurfaceTemperatureVis = { 
  min: 15,
  max: 35,
  palette: [
    '040274', '040281', '0502a3', '0502b8', '0502ce', '0502e6',
    '0602ff', '235cb1', '307ef3', '269db1', '30c8e2', '32d3ef',
    '3be285', '3ff38f', '86e26f', '3ae237', 'b5e22e', 'd6e21f',
    'fff705', 'ffd611', 'ffb613', 'ff8b13', 'ff6e08', 'ff500d',
    'ff0000', 'de0101', 'c21301', 'a71001', '911003'
  ],
};

var vizUHI = {
  min: -5,
  max: 5,
  palette: ['2c7bb6','abd9e9','ffffbf','fdae61','d7191c']
};
var vizUHIKelas = {
  min: 0,
  max: 3,
  palette: ['abd9e9','ffffbf','fdae61','d7191c']
  //palette: ['blue','green','yellow','red']
};
var vizUHIKelas2 = {
  min: 0,
  max: 5,
  palette: ['2c7bb6','abd9e9','ffffbf','fdae61','d7191c']
};
function scale(image){
  var lstcelcius = image.multiply(0.02).subtract(273.15);
  return image.addBands(lstcelcius, null, true)
}

var Indo = FAO0.filter(ee.Filter.inList('ADM0_NAME',['Indonesia']));
var Kota = ADMProv.filter(ee.Filter.inList('NAME_1',['Jawa Tengah']));

var today = ee.Date(new Date().toISOString().split('T')[0].slice(0,10));
var today30 = ee.Date(new Date().toISOString().split('T')[0].slice(0,10)).advance(-30, 'day');

var dataset = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(Indo)
                  .filter(ee.Filter.date(today30, today))
                  .map(scale);
                  
var Daytime = dataset.select('LST_Day_1km').mean().clip(Indo);
var Nighttime = dataset.select('LST_Night_1km').mean().clip(Indo);

Map.centerObject(Indo,4)
Map.addLayer(Daytime,landSurfaceTemperatureVis,'LST Day Monthly');
Map.addLayer(Nighttime,landSurfaceTemperatureVis,'LST Night Monthly',false);

var LSTMean = Daytime.reduceRegion(ee.Reducer.mean(), Indo, 1000).get('LST_Day_1km');
var LSTstdDev = Daytime.reduceRegion(ee.Reducer.stdDev(), Indo, 1000).get('LST_Day_1km');

var UHI = Daytime.expression('LST-(a+(0.5*b))',{
    'LST':Daytime,
    'a':ee.Number(LSTMean), 
    'b':ee.Number(LSTstdDev),
  }).rename('UHI');
Map.addLayer(UHI,vizUHI,'UHI',false);

var Kelas = ee.Image(1)
          .where(UHI.lt(0), 0)
          .where(UHI.gte(0).and(UHI.lt(2)), 1)
          .where(UHI.gte(2).and(UHI.lt(4)), 2)
          .where(UHI.gte(4), 3)
          .clip(Indo);
Map.addLayer(Kelas, vizUHIKelas, 'UHI Class',false);

//CHART
var today360 = ee.Date(new Date().toISOString().split('T')[0]).advance(-365, 'day');

var datasetChart = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(Kota)
                  .filter(ee.Filter.date('2024-01-01', '2024-05-01'))
                  .map(scale);
                  
var chart = ui.Chart.image.seriesByRegion({
      imageCollection: datasetChart,  
      regions: Kota,      
      reducer: ee.Reducer.mean(),   
      band: 'LST_Day_1km',
      scale: 10000,
      xProperty: 'system:time_start',
      seriesProperty: 'system:index'
    }).setSeriesNames(['Suhu']).setChartType('LineChart');
      chart.setOptions({
      title: 'Rata-Rata Suhu Permukaan di Indonesia',
      //trendlines: {0: {showR2: true, visibleInLegend: true}}
});

//CHART LUASAN
var area_chart = ui.Chart.image.byClass({
  image: ee.Image.pixelArea().divide(1e6).addBands(Kelas) , 
  classBand: 'constant' , 
  region: Indo, 
  reducer: ee.Reducer.sum(), 
  scale: 1000, 
  classLabels:['Non UHI','Low','Moderate','High']
  })
  .setOptions({title: 'Urban Heat Island Area in Indonesia (km2)',colors: ['abd9e9','ffffbf','fdae61','d7191c']});

//CHART RERATA LST PER PROVINSI
var ProvIDN = FAO1.filter(ee.Filter.eq('ADM0_NAME', 'Indonesia'));
function getMeanLST(feature) {
  var mean_day = datasetChart.mean().select('LST_Day_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Day_1km');
  var mean_night = datasetChart.mean().select('LST_Night_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Night_1km');
  return feature.set({
    'LST_Day_1km': mean_day,
    'LST_Night_1km': mean_night
  });
}
var province_means = ProvIDN.map(getMeanLST);
var chartMeanProv = ui.Chart.feature.byFeature({
  features: province_means,
  xProperty: 'ADM1_NAME',
  yProperties: ['LST_Day_1km', 'LST_Night_1km']
}).setOptions({
  title: 'Mean LST Monthly Daytime and Nighttime per Province',
  hAxis: {title: 'Provinsi'},
  vAxis: {title: 'LST (°C)'},
  series: {
    0: {color: 'red', label: 'LST Siang'},
    1: {color: 'blue', label: 'LST Malam'}
  },
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART RERATA UHI PER PROVINSI
function getMeanUHI(feature) {
  var urban_heat = UHI.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('UHI');
  return feature.set({'UHI': urban_heat,});
}
var province_meanUHI = ProvIDN.map(getMeanUHI);
var chartMeanUHIProv = ui.Chart.feature.byFeature({
  features: province_meanUHI,
  xProperty: 'ADM1_NAME',
  yProperties: ['UHI']
}).setOptions({
  title: 'Mean UHI Intensity Monthly Daytime per Province',
  hAxis: {title: 'Province'},
  vAxis: {title: 'UHI Intensity (°C)'},
  series: {0: {color: 'fdae61', label: 'UHI'},},
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART LUASAN UHI PER PROVINSI
var luasUHI1= Kelas.eq(1).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: ProvIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI2 = Kelas.eq(2).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: ProvIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI3 = Kelas.eq(3).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: ProvIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var gabungData = luasUHI1.map(function(feature){
var provinsi = feature.get('ADM1_NAME');
var luasUHI1= ee.Number(feature.get('sum'));
var luasUHI2_2 = ee.Number(luasUHI2.filter(ee.Filter.eq('ADM1_NAME', provinsi)).first().get('sum'));
var luasUHI3_3 = ee.Number(luasUHI3.filter(ee.Filter.eq('ADM1_NAME', provinsi)).first().get('sum'));
  return ee.Feature(null, {
    'ADM1_NAME': provinsi,
    'Low': luasUHI1,
    'Moderate': luasUHI2_2,
    'High': luasUHI3_3
    });
  });
var AreaChartUHI = ui.Chart.feature.byFeature({
  features: gabungData,
  xProperty: 'ADM1_NAME',
  yProperties: ['Low', 'Moderate','High']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Luas',
    hAxis: {title: 'Province'},
    vAxis: {title: 'Area Km2'},
    series: {
    0: {color: 'abd9e9', label: 'Low'},
    1: {color: 'fdae61', label: 'Moderate'},
    2: {color: 'd7191c', label: 'High'}
  },
  });

//CHART POPULATION UHI4
var imagePop = ee.Image("WorldPop/GP/100m/pop_age_sex/IDN_2020").select('population');
var maskPop = imagePop.updateMask(Kelas.eq(3));
var jumlahPop = maskPop.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: Indo,
  scale: 1000,
  maxPixels: 1e9,
  bestEffort: true,
}).get('population').getInfo();
var angkapixel = ee.Number(jumlahPop).round().getInfo();  
var popLabel = ui.Label({
  value: 'Estimated Population Affected by UHI (High): ' + angkapixel + ' People'
});
//CHART POPULATION UHI4 PER PROVINCE
var TotalPop = imagePop.updateMask(Kelas.eq(3)).rename('Jumlah')
    .reduceRegions({
      collection: ProvIDN,
      reducer: ee.Reducer.sum(),
      scale: 1000, 
      crs: 'EPSG:4326',
      tileScale: 2
  });
var TotalProv = TotalPop.map(function(feature){
var Provinsi = ee.String(feature.get('ADM1_NAME'));
var Jml = ee.Number(feature.get('sum'));
  return ee.Feature(null, {'ADM1_NAME': Provinsi, 'Jumlah': Jml});
});
var ChartTotalPop = ui.Chart.feature.byFeature({
  features: TotalProv,
  xProperty: 'ADM1_NAME',
  yProperties: ['Jumlah']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Estimated Population Affected by Province',
    hAxis: {title: 'Province'},
    vAxis: {title: 'Population'}
  });

//LEGENDA
//POSISI PANEL LEGENDA
var panel = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '5px;'
  }
});

var title = ui.Label({
  value: 'Legend',
  style: {
    fontWeight: 'bold',
  }
});
panel.add(title);

var color = ['abd9e9','ffffbf','fdae61','d7191c'];
var lc_class = ['Non UHI','Low','Moderate','High'];

var list_legend = function(color, description) {
  var c = ui.Label({
    style: {
      backgroundColor: color,
      padding: '10px',
      margin: '4px'
    }
  })
  var ds = ui.Label({
    value: description,
    style: {
      margin: '5px'
    }
  })
  return ui.Panel({
    widgets: [c, ds],
    layout: ui.Panel.Layout.Flow('horizontal')
  })
}
for(var a = 0; a < 4; a++){
  panel.add(list_legend(color[a], lc_class[a]))
}
//LEGEND BEDROCK OXIDE SAPROLITE
// MEMBUAT JUDUL LABEL
var LSTTitle = ui.Label({
  value: 'LST',
  //style: {fontWeight: 'bold'}
});
var UHITitle = ui.Label({
  value: 'UHI',
  //style: {fontWeight: 'bold'}
});
//create the legend image
var lon = ee.Image.pixelLonLat().select('longitude');
var gradient = lon;
var legendImage = gradient.visualize(landSurfaceTemperatureVis);
var legendImage2 = gradient.visualize(vizUHI);

//create text on top of legend
var labelLow = ui.Panel({
    widgets: [ui.Label('15°C')
    ],
});
var labelLow2 = ui.Panel({
    widgets: [ui.Label('-5°C')
    ],
});

//create thumbnail from the image
var thumbnail = ui.Thumbnail ({
  image: legendImage,
  params: {bbox:'0,0,60,60',dimensions: '170x15'},
  style: {padding: '1px', position: 'top-center'}
});
var thumbnail2 = ui.Thumbnail ({
  image: legendImage2,
  params: {bbox:'0,0,20,20',dimensions: '170x15'},
  style: {padding: '1px', position: 'top-center'}
});

//create text on top of legend
var labelHigh = ui.Panel({
    widgets: [ui.Label('35°C')
    ],
});
var labelHigh2 = ui.Panel({
    widgets: [ui.Label('5°C')
    ],
});

var LSTLegend = ui.Panel({
  layout: ui.Panel.Layout.flow('horizontal'),
  widgets: [LSTTitle, labelLow, thumbnail, labelHigh],
  style: {position: 'bottom-left',padding: '8px 4px'}});
var UHILegend = ui.Panel({
  layout: ui.Panel.Layout.flow('horizontal'),
  widgets: [UHITitle, labelLow2, thumbnail2, labelHigh2],
  style: {position: 'bottom-left',padding: '8px 4px'}});

var LegendAll = ui.Panel({
  widgets: [LSTLegend, UHILegend],
  });
panel.add(LegendAll)

        //--------------------------------------
                    //FOR REGION OF INTEREST
        //--------------------------------------
        
        
//PANEL
var mainPanel = ui.Panel({
  layout: ui.Panel.Layout.flow('vertical', true),
  style: {width: '380px', padding: '20px'}
});

var chartmainPanel = ui.Panel({
  layout: ui.Panel.Layout.flow('vertical', true),
  style: {width: '250px', padding: '20px'}
});
var datePanel = ui.Panel({
  layout: ui.Panel.Layout.flow('horizontal', true)
}); 

var paramPanel = ui.Panel({
  layout: ui.Panel.Layout.flow('horizontal', true)
});
var typePanel = ui.Panel({
  layout: ui.Panel.Layout.flow('vertical', true)
});
var SelectPanel = ui.Panel({
  layout: ui.Panel.Layout.flow('horizontal', true)
});

var chartPanel = ui.Panel();
var chartPanel2 = ui.Panel();
var koordPanel = ui.Panel();

ui.root.add(mainPanel);
ui.root.insert(0, chartmainPanel);

//LABEL
var Title1=ui.Label({
  value:'Urban Heat Island Explorer',
  style:{color:'ca0020', fontSize:'20px', fontWeight:'bold',textAlign:'center', margin:'10px 10px 0px 55px'},
});
var Title2=ui.Label({
  value:'Discover the Urban Heat Phenomenon in Indonesia Using Big Data Geospatial Cloud Computing Platform',
  style:{color:'black', fontSize:'14px', fontWeight:'bold',textAlign:'center', margin:'0px 10px 10px 10px'},
});
var selectLabel = ui.Label({
  value: 'Select Region of Interest',
});
var chartInfoLabel = ui.Label({
  value: 'Information Chart',
  style:{fontWeight:'bold'},
});
var PopInfoLabel = ui.Label({
  value: 'Population Affected Information',
  style:{fontWeight:'bold'},
});
var AddInfoLabel = ui.Label({
  value: 'Additional Information',
  style:{fontWeight:'bold'},
});

//BUTTON
var ProvButton = ui.Button({
  label: 'Province',
  onClick: ExtractProv,
  style: {width: '100px'}
});
var RegButton = ui.Button({
  label: 'Regency/City',
  onClick: ExtractCity,
  style: {width: '100px'}
});
var startAOI = ui.Button({
  label: 'START',
  //onClick: MulaiAOI,
  style: {width: '100px'}
});
SelectPanel.add(ProvButton);
SelectPanel.add(RegButton);


//DATE
var dateLabel = ui.Label({
  value: 'Select Date (Year-Month-Day)',
  style: {fontWeight: 'bold'}
});

var startDate = ui.Textbox({
  value: '2024-04-01',
  style: {width: '100px',border:'1px solid black'}
});

var endDate = ui.Textbox({
  value: '2024-04-30',
  style: {width: '100px',border:'1px solid black'}
});

var hubungLabel = ui.Label({value: '-',style: {width: '5px'}});
datePanel.add(startDate);
datePanel.add(hubungLabel);
datePanel.add(endDate);


    mainPanel.clear();
    mainPanel.add(Title1);
    mainPanel.add(Title2);
    mainPanel.add(selectLabel); 
    mainPanel.add(SelectPanel);
    mainPanel.add(panel)
    chartmainPanel.add(chartInfoLabel);
    chartPanel.add(area_chart);
    chartPanel.add(chartMeanProv);
    chartPanel.add(chartMeanUHIProv);
    chartPanel.add(AreaChartUHI);
    chartPanel.add(PopInfoLabel);
    chartPanel.add(popLabel);
    chartPanel.add(ChartTotalPop);
    chartmainPanel.add(chartPanel);    

//-------------------------------------------
        //REGION OF PROVINCE
//-----------------------------------------
var provText = ui.Textbox({ //Sample city
  value: 'Jawa Tengah',
  style: {width: '100px',border:'1px solid black'}
});
var provLabel = ui.Label({
  value: 'Name of the Province',
  style: {fontWeight: 'bold'}
});
var startProv = ui.Button({
  label: 'START',
  onClick: MulaiProv,
  style: {width: '100px'}
});

function ExtractProv(){
  typePanel.clear();
  mainPanel.clear();
  typePanel.add(provLabel);
  typePanel.add(provText);
    mainPanel.add(Title1);
    mainPanel.add(Title2);
    mainPanel.add(selectLabel); 
    mainPanel.add(SelectPanel);
    mainPanel.add(typePanel);
    mainPanel.add(dateLabel);
    mainPanel.add(datePanel);
    mainPanel.add(startProv);
    mainPanel.add(panel)
}

                          //FUNCTION MULAI PROV
function MulaiProv(){
  Map.layers().reset();
  chartmainPanel.clear();
  chartPanel.clear();
  chartPanel2.clear();
  chartmainPanel.add(chartInfoLabel);
  
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameProv = provText.getValue();
  var aoi = ADMProv.filter(ee.Filter.inList('NAME_1',[nameProv]));
  
function scale(image){
  var lstcelcius = image.multiply(0.02).subtract(273.15);
  return image.addBands(lstcelcius, null, true)
}

  var dataset = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
                  
var Daytime = dataset.select('LST_Day_1km').mean().clip(aoi);
var Nighttime = dataset.select('LST_Night_1km').mean().clip(aoi);

Map.centerObject(aoi,8)
Map.addLayer(Daytime,landSurfaceTemperatureVis,'LST Day Monthly');
Map.addLayer(Nighttime,landSurfaceTemperatureVis,'LST Night Monthly',false);

var LSTMean = Daytime.reduceRegion(ee.Reducer.mean(), aoi, 1000).get('LST_Day_1km');
var LSTstdDev = Daytime.reduceRegion(ee.Reducer.stdDev(), aoi, 1000).get('LST_Day_1km');

var UHI = Daytime.expression('LST-(a+(0.5*b))',{
    'LST':Daytime,
    'a':ee.Number(LSTMean),
    'b':ee.Number(LSTstdDev),
  }).rename('UHI');
Map.addLayer(UHI,vizUHI,'UHI',false);

var Kelas = ee.Image(1)
          .where(UHI.lt(0), 0)
          .where(UHI.gte(0).and(UHI.lt(2)), 1)
          .where(UHI.gte(2).and(UHI.lt(4)), 2)
          .where(UHI.gte(4), 3)
          .clip(aoi);
Map.addLayer(Kelas, vizUHIKelas, 'UHI Class',false);

  //CHART
//LST PROV
    //Function UHI
  var indicesUHI = function(img){
    var LSTMean = img.select('LST_Day_1km').reduceRegion(ee.Reducer.mean(), aoi, 1000).get('LST_Day_1km');
    var LSTstdDev = img.select('LST_Day_1km').reduceRegion(ee.Reducer.stdDev(), aoi, 1000).get('LST_Day_1km');

    var UHI = img.select('LST_Day_1km').expression('LST-(a+(0.5*b))',{
    'LST':img.select('LST_Day_1km'),
    'a':ee.Number(LSTMean),
    'b':ee.Number(LSTstdDev),
    }).rename('UHI');
      return img.addBands(UHI, null, true);
  };
    
  
var datasetTimeSeries = ee.ImageCollection('MODIS/061/MYD11A2').filterBounds(aoi)
                  .filter(ee.Filter.date('2024-01-01', end))
                  .map(scale)
                  .map(indicesUHI);
                  
var chart = ui.Chart.image.seriesByRegion({
      imageCollection: datasetTimeSeries,  
      regions: aoi,      
      reducer: ee.Reducer.mean(),   
      band: 'LST_Day_1km',
      scale: 10000,
      xProperty: 'system:time_start',
      seriesProperty: 'system:index'
    }).setSeriesNames(['Suhu']).setChartType('LineChart');
      chart.setOptions({
      title: 'Mean LST Daylight Temperature',
      trendlines: {0: {showR2: true, visibleInLegend: false}}
});
var chartUHI = ui.Chart.image.seriesByRegion({
      imageCollection: datasetTimeSeries,  
      regions: aoi,      
      reducer: ee.Reducer.mean(),   
      band: 'UHI',
      scale: 10000,
      xProperty: 'system:time_start',
      seriesProperty: 'system:index'
    }).setSeriesNames(['Suhu']).setChartType('LineChart');
      chartUHI.setOptions({
      title: 'Mean UHI Value',
      trendlines: {0: {showR2: true, visibleInLegend: false}}
});

//CHART LUASAN
var area_chart = ui.Chart.image.byClass({
  image: ee.Image.pixelArea().divide(1e6).addBands(Kelas) , 
  classBand: 'constant' , 
  region: aoi, 
  reducer: ee.Reducer.sum(), 
  scale: 1000, 
  classLabels:['Non UHI','Low','Moderate','High']
  })
  .setOptions({title: 'Urban Heat Island Area (km2)',colors: ['abd9e9','ffffbf','fdae61','d7191c']});

//CHART RERATA LST PER KABUPATEN
var datasetChart = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
var KabIDN = ADMKab.filter(ee.Filter.inList('NAME_1', [nameProv]));
function getMeanLST(feature) {
  var mean_day = datasetChart.mean().select('LST_Day_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Day_1km');
  var mean_night = datasetChart.mean().select('LST_Night_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Night_1km');
  return feature.set({
    'LST_Day_1km': mean_day,
    'LST_Night_1km': mean_night
  });
}
var kabkot_means = KabIDN.map(getMeanLST);
var chartMeanKab = ui.Chart.feature.byFeature({
  features: kabkot_means,
  xProperty: 'NAME_2',
  yProperties: ['LST_Day_1km', 'LST_Night_1km']
}).setOptions({
  title: 'Mean LST Monthly Daytime and Nighttime per Regency/City',
  hAxis: {title: 'Regency/City'},
  vAxis: {title: 'LST (°C)'},
  series: {
    0: {color: 'red', label: 'LST Day'},
    1: {color: 'blue', label: 'LST Night'}
  },
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART RERATA UHI PER KABKOT
function getMeanUHI(feature) {
  var urban_heat = UHI.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('UHI');
  return feature.set({'UHI': urban_heat,});
}
var kabkot_meanUHI = KabIDN.map(getMeanUHI);
var chartMeanUHIKab = ui.Chart.feature.byFeature({
  features: kabkot_meanUHI,
  xProperty: 'NAME_2',
  yProperties: ['UHI']
}).setOptions({
  title: 'Mean UHI Intensity Monthly Daytime per Regency/City',
  hAxis: {title: 'Regency/City'},
  vAxis: {title: 'UHI Intensity (°C)'},
  series: {0: {color: 'fdae61', label: 'UHI'},},
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART LUASAN UHI PER PROVINSI
var luasUHI1= Kelas.eq(1).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KabIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI2 = Kelas.eq(2).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KabIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI3 = Kelas.eq(3).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KabIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var gabungData = luasUHI1.map(function(feature){
var kabkot = feature.get('NAME_2');
var luasUHI1= ee.Number(feature.get('sum'));
var luasUHI2_2 = ee.Number(luasUHI2.filter(ee.Filter.eq('NAME_2', kabkot)).first().get('sum'));
var luasUHI3_3 = ee.Number(luasUHI3.filter(ee.Filter.eq('NAME_2', kabkot)).first().get('sum'));
  return ee.Feature(null, {
    'NAME_2': kabkot,
    'Low': luasUHI1,
    'Moderate': luasUHI2_2,
    'High': luasUHI3_3
    });
  });
var AreaChartUHI = ui.Chart.feature.byFeature({
  features: gabungData,
  xProperty: 'NAME_2',
  yProperties: ['Low', 'Moderate','High']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Area of UHI Class',
    hAxis: {title: 'Regency/City'},
    vAxis: {title: 'Area Km2'},
    series: {
    0: {color: 'abd9e9', label: 'Low'},
    1: {color: 'fdae61', label: 'Moderate'},
    2: {color: 'd7191c', label: 'High'}
  },
  });
  
//CHART POPULATION UHI4
var imagePop = ee.Image("WorldPop/GP/100m/pop_age_sex/IDN_2020").select('population');
var maskPop = imagePop.updateMask(Kelas.eq(3));
var jumlahPop = maskPop.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: aoi,
  scale: 1000,
  maxPixels: 1e9,
  bestEffort: true,
}).get('population').getInfo();
var angkapixel = ee.Number(jumlahPop).round().getInfo();  
var popLabel = ui.Label({
  value: 'Estimated Population Affected by UHI (High): ' + angkapixel + ' People'
});
//CHART POPULATION UHI4 PER PROVINCE
var TotalPop = imagePop.updateMask(Kelas.eq(3)).rename('Jumlah')
    .reduceRegions({
      collection: KabIDN,
      reducer: ee.Reducer.sum(),
      scale: 1000, 
      crs: 'EPSG:4326',
      tileScale: 2
  });
var TotalKab = TotalPop.map(function(feature){
var Kabkot = ee.String(feature.get('NAME_2'));
var Jml = ee.Number(feature.get('sum'));
  return ee.Feature(null, {'NAME_2': Kabkot, 'Jumlah': Jml});
});
var ChartTotalPop = ui.Chart.feature.byFeature({
  features: TotalKab,
  xProperty: 'NAME_2',
  yProperties: ['Jumlah']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Estimated Population Affected per Regency/City',
    hAxis: {title: 'Regency/City'},
    vAxis: {title: 'Population'}
  });

//INSPECTOR
//POSISI PANEL
var inspectPanel = ui.Panel({
  style: {
    position: 'bottom-right',
    padding: '4px 3px'
  }
});

var lon = ui.Label()
var lat = ui.Label()
//Define callback function
function showValue(value) {
  var judulLabel = ui.Label({
    value: 'Land Surface Temperature Value',
    style: {
      fontWeight: 'bold',
    }
  });
  var valueLabel = ui.Label(value);

  inspectPanel.clear();
  inspectPanel.add(judulLabel);
  inspectPanel.add(valueLabel);
}

function inspect(coords) {
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameProv = provText.getValue();
  var aoi = ADMProv.filter(ee.Filter.inList('NAME_1',[nameProv]));
  //DATASET
  var imageLST = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale)
                  .mean()
                  .clip(aoi);
  var point = ee.Geometry.Point(coords.lon, coords.lat);
  var konsentrat = imageLST.select('LST_Day_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: point,
    scale: Map.getScale()
  }).get('LST_Day_1km');

  konsentrat.evaluate(showValue);  
}

Map.onClick(inspect);

koordPanel.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));
Map.onClick(function(coords2) {
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameProv = provText.getValue();
  var aoi = ADMProv.filter(ee.Filter.inList('NAME_1',[nameProv]));
  //DATASET
  var imagesChart = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
  lon.setValue('Lat.: ' + coords2.lon.toFixed(5)),
  lat.setValue('Long.: ' + coords2.lat.toFixed(5));
  var point = ee.Geometry.Point(coords2.lon, coords2.lat);
  var dot = ui.Map.Layer(point);
  //Map.layers().set(1, dot);
  chartPanel2.clear();
    var chartz = ui.Chart.image.seriesByRegion({
      imageCollection: imagesChart,
      regions: point,
      reducer: ee.Reducer.mean(),
      band: 'LST_Day_1km',
      scale: 30,
      xProperty: 'system:time_start'
    }).setSeriesNames(['LST']).setChartType('LineChart');
      chartz.setOptions({
      title: 'LST Value by Specific Location',
      trendlines: {0: {showR2: true, visibleInLegend: true}}
});
chartPanel2.add(chartz);
})

    chartPanel.add(chart);
    chartPanel.add(chartUHI);
    chartPanel.add(area_chart);
    chartPanel.add(chartMeanKab);
    chartPanel.add(chartMeanUHIKab);
    chartPanel.add(AreaChartUHI);
    //chartPanel.add(PopInfoLabel);
    chartPanel.add(popLabel);
    chartPanel.add(ChartTotalPop);
    chartmainPanel.add(chartPanel); 
    mainPanel.add(AddInfoLabel);
    mainPanel.add(koordPanel);
    mainPanel.add(inspectPanel);
    mainPanel.add(chartPanel2);
} //END OF FUNCTION

//-------------------------------------------
        //REGION OF REGENCY/CITY
//-----------------------------------------
var cityText = ui.Textbox({ //Sample city
  value: 'Kota Semarang',
  style: {width: '100px',border:'1px solid black'}
});
var cityLabel = ui.Label({
  value: 'Name of the Regency/City',
  style: {fontWeight: 'bold'}
});
var startCity = ui.Button({
  label: 'START',
  onClick: MulaiCity,
  style: {width: '100px'}
});

function ExtractCity(){
  typePanel.clear();
  mainPanel.clear();
  koordPanel.clear();
  typePanel.add(cityLabel);
  typePanel.add(cityText);
    mainPanel.add(Title1);
    mainPanel.add(Title2);
    mainPanel.add(selectLabel); 
    mainPanel.add(SelectPanel);
    mainPanel.add(typePanel);
    mainPanel.add(dateLabel);
    mainPanel.add(datePanel);
    mainPanel.add(startCity);
    mainPanel.add(panel)
}

                          //FUNCTION MULAI KABKOT
function MulaiCity(){
  Map.layers().reset();
  chartmainPanel.clear();
  chartPanel.clear();
  chartPanel2.clear();
  koordPanel.clear();
  chartmainPanel.add(chartInfoLabel);
  
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameCity = cityText.getValue();
  var aoi = ADMKab.filter(ee.Filter.inList('NAME_2',[nameCity]));
  
function scale(image){
  var lstcelcius = image.multiply(0.02).subtract(273.15);
  return image.addBands(lstcelcius, null, true)
}

  var dataset = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
                  
var Daytime = dataset.select('LST_Day_1km').mean().clip(aoi);
var Nighttime = dataset.select('LST_Night_1km').mean().clip(aoi);

Map.centerObject(aoi)
Map.addLayer(Daytime,landSurfaceTemperatureVis,'LST Day Monthly');
Map.addLayer(Nighttime,landSurfaceTemperatureVis,'LST Night Monthly',false);

var LSTMean = Daytime.reduceRegion(ee.Reducer.mean(), aoi, 1000).get('LST_Day_1km');
var LSTstdDev = Daytime.reduceRegion(ee.Reducer.stdDev(), aoi, 1000).get('LST_Day_1km');

var UHI = Daytime.expression('LST-(a+(0.5*b))',{
    'LST':Daytime,
    'a':ee.Number(LSTMean),
    'b':ee.Number(LSTstdDev),
  }).rename('UHI');
Map.addLayer(UHI,vizUHI,'UHI',false);

var Kelas = ee.Image(1)
          .where(UHI.lt(0), 0)
          .where(UHI.gte(0).and(UHI.lt(2)), 1)
          .where(UHI.gte(2).and(UHI.lt(4)), 2)
          .where(UHI.gte(4), 3)
          .clip(aoi);
Map.addLayer(Kelas, vizUHIKelas, 'UHI Class',false);

  //CHART
//LST PROV
    //Function UHI
  var indicesUHI = function(img){
    var LSTMean = img.select('LST_Day_1km').reduceRegion(ee.Reducer.mean(), aoi, 1000).get('LST_Day_1km');
    var LSTstdDev = img.select('LST_Day_1km').reduceRegion(ee.Reducer.stdDev(), aoi, 1000).get('LST_Day_1km');

    var UHI = img.select('LST_Day_1km').expression('LST-(a+(0.5*b))',{
    'LST':img.select('LST_Day_1km'),
    'a':ee.Number(LSTMean),
    'b':ee.Number(LSTstdDev),
    }).rename('UHI');
      return img.addBands(UHI, null, true);
  };
    
  
var datasetTimeSeries = ee.ImageCollection('MODIS/061/MYD11A2').filterBounds(aoi)
                  .filter(ee.Filter.date('2024-01-01', end))
                  .map(scale)
                  .map(indicesUHI);
                  
var chart = ui.Chart.image.seriesByRegion({
      imageCollection: datasetTimeSeries,  
      regions: aoi,      
      reducer: ee.Reducer.mean(),   
      band: 'LST_Day_1km',
      scale: 10000,
      xProperty: 'system:time_start',
      seriesProperty: 'system:index'
    }).setSeriesNames(['Suhu']).setChartType('LineChart');
      chart.setOptions({
      title: 'Mean LST Daylight Temperature',
      trendlines: {0: {showR2: true, visibleInLegend: false}}
});
var chartUHI = ui.Chart.image.seriesByRegion({
      imageCollection: datasetTimeSeries,  
      regions: aoi,      
      reducer: ee.Reducer.mean(),   
      band: 'UHI',
      scale: 10000,
      xProperty: 'system:time_start',
      seriesProperty: 'system:index'
    }).setSeriesNames(['Suhu']).setChartType('LineChart');
      chartUHI.setOptions({
      title: 'Mean UHI Value',
      trendlines: {0: {showR2: true, visibleInLegend: false}}
});

//CHART LUASAN
var area_chart = ui.Chart.image.byClass({
  image: ee.Image.pixelArea().divide(1e6).addBands(Kelas) , 
  classBand: 'constant' , 
  region: aoi, 
  reducer: ee.Reducer.sum(), 
  scale: 1000, 
  classLabels:['Non UHI','Low','Moderate','High']
  })
  .setOptions({title: 'Urban Heat Island Area (km2)',colors: ['abd9e9','ffffbf','fdae61','d7191c']});

//CHART RERATA LST PER KECAMATAN
var datasetChart = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
var KecIDN = ADMKEC.filter(ee.Filter.inList('NAME_2', [nameCity]));
function getMeanLST(feature) {
  var mean_day = datasetChart.mean().select('LST_Day_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Day_1km');
  var mean_night = datasetChart.mean().select('LST_Night_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('LST_Night_1km');
  return feature.set({
    'LST_Day_1km': mean_day,
    'LST_Night_1km': mean_night
  });
}
var Kec_means = KecIDN.map(getMeanLST);
var chartMeanKec = ui.Chart.feature.byFeature({
  features: Kec_means,
  xProperty: 'NAME_3',
  yProperties: ['LST_Day_1km', 'LST_Night_1km']
}).setOptions({
  title: 'Mean LST Monthly Daytime and Nighttime per District',
  hAxis: {title: 'District'},
  vAxis: {title: 'LST (°C)'},
  series: {
    0: {color: 'red', label: 'LST Day'},
    1: {color: 'blue', label: 'LST Night'}
  },
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART RERATA UHI PER KABKOT
function getMeanUHI(feature) {
  var urban_heat = UHI.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: feature.geometry(),
    scale: 10000
  }).get('UHI');
  return feature.set({'UHI': urban_heat,});
}
var Kec_meanUHI = KecIDN.map(getMeanUHI);
var chartMeanUHIKec = ui.Chart.feature.byFeature({
  features: Kec_meanUHI,
  xProperty: 'NAME_3',
  yProperties: ['UHI']
}).setOptions({
  title: 'Mean UHI Intensity Monthly Daytime per District',
  hAxis: {title: 'District'},
  vAxis: {title: 'UHI Intensity (°C)'},
  series: {0: {color: 'fdae61', label: 'UHI'},},
  legend: {position: 'bottom'}
}).setChartType('ColumnChart');

//CHART LUASAN UHI PER PROVINSI
var luasUHI1= Kelas.eq(1).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KecIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI2 = Kelas.eq(2).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KecIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var luasUHI3 = Kelas.eq(3).multiply(ee.Image.pixelArea()).divide(10000).reduceRegions({
  collection: KecIDN,
  reducer: ee.Reducer.sum(),
  scale: 10000,
  crs: 'EPSG:4326'
  });
var gabungData = luasUHI1.map(function(feature){
var Kec = feature.get('NAME_3');
var luasUHI1= ee.Number(feature.get('sum'));
var luasUHI2_2 = ee.Number(luasUHI2.filter(ee.Filter.eq('NAME_3', Kec)).first().get('sum'));
var luasUHI3_3 = ee.Number(luasUHI3.filter(ee.Filter.eq('NAME_3', Kec)).first().get('sum'));
  return ee.Feature(null, {
    'NAME_3': Kec,
    'Low': luasUHI1,
    'Moderate': luasUHI2_2,
    'High': luasUHI3_3
    });
  });
var AreaChartUHI = ui.Chart.feature.byFeature({
  features: gabungData,
  xProperty: 'NAME_3',
  yProperties: ['Low', 'Moderate','High']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Area of UHI Class',
    hAxis: {title: 'District'},
    vAxis: {title: 'Area Km2'},
    series: {
    0: {color: 'abd9e9', label: 'Low'},
    1: {color: 'fdae61', label: 'Moderate'},
    2: {color: 'd7191c', label: 'High'}
  },
  });
  
//CHART POPULATION UHI4
var imagePop = ee.Image("WorldPop/GP/100m/pop_age_sex/IDN_2020").select('population');
var maskPop = imagePop.updateMask(Kelas.eq(3));
var jumlahPop = maskPop.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: aoi,
  scale: 1000,
  maxPixels: 1e9,
  bestEffort: true,
}).get('population').getInfo();
var angkapixel = ee.Number(jumlahPop).round().getInfo();  
var popLabel = ui.Label({
  value: 'Estimated Population Affected by UHI (High): ' + angkapixel + ' People'
});
//CHART POPULATION UHI4 PER PROVINCE
var TotalPop = imagePop.updateMask(Kelas.eq(3)).rename('Jumlah')
    .reduceRegions({
      collection: KecIDN,
      reducer: ee.Reducer.sum(),
      scale: 1000, 
      crs: 'EPSG:4326',
      tileScale: 2
  });
var TotalKec = TotalPop.map(function(feature){
var Kec = ee.String(feature.get('NAME_3'));
var Jml = ee.Number(feature.get('sum'));
  return ee.Feature(null, {'NAME_3': Kec, 'Jumlah': Jml});
});
var ChartTotalPop = ui.Chart.feature.byFeature({
  features: TotalKec,
  xProperty: 'NAME_3',
  yProperties: ['Jumlah']
  }).setChartType('ColumnChart')
  .setOptions({
    title: 'Estimated Population Affected per Regency/City',
    hAxis: {title: 'Regency/City'},
    vAxis: {title: 'Population'}
  });

//INSPECTOR
//POSISI PANEL
var inspectPanel = ui.Panel({
  style: {
    position: 'bottom-right',
    padding: '4px 3px'
  }
});

var lon = ui.Label()
var lat = ui.Label()
//Define callback function
function showValue(value) {
  var judulLabel = ui.Label({
    value: 'Land Surface Temperature Value',
    style: {
      fontWeight: 'bold',
    }
  });
  var valueLabel = ui.Label(value);

  inspectPanel.clear();
  inspectPanel.add(judulLabel);
  inspectPanel.add(valueLabel);
}

function inspect(coords) {
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameCity = cityText.getValue();
  var aoi = ADMKab.filter(ee.Filter.inList('NAME_2',[nameCity]));
  //DATASET
  var imageLST = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale)
                  .mean()
                  .clip(aoi);
  var point = ee.Geometry.Point(coords.lon, coords.lat);
  var konsentrat = imageLST.select('LST_Day_1km').reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: point,
    scale: Map.getScale()
  }).get('LST_Day_1km');

  konsentrat.evaluate(showValue);  
}

Map.onClick(inspect);

koordPanel.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));
Map.onClick(function(coords2) {
  var start = startDate.getValue();
  var end = endDate.getValue();
  var nameCity = cityText.getValue();
  var aoi = ADMKab.filter(ee.Filter.inList('NAME_2',[nameCity]));
  //DATASET
  var imagesChart = ee.ImageCollection('MODIS/061/MOD11A1').filterBounds(aoi)
                  .filter(ee.Filter.date(start, end))
                  .map(scale);
  lon.setValue('Lat.: ' + coords2.lon.toFixed(5)),
  lat.setValue('Long.: ' + coords2.lat.toFixed(5));
  var point = ee.Geometry.Point(coords2.lon, coords2.lat);
  var dot = ui.Map.Layer(point);
  //Map.layers().set(1, dot);
  chartPanel2.clear();
    var chartz = ui.Chart.image.seriesByRegion({
      imageCollection: imagesChart,
      regions: point,
      reducer: ee.Reducer.mean(),
      band: 'LST_Day_1km',
      scale: 30,
      xProperty: 'system:time_start'
    }).setSeriesNames(['LST']).setChartType('LineChart');
      chartz.setOptions({
      title: 'LST Value by Specific Location',
      trendlines: {0: {showR2: true, visibleInLegend: true}}
});
chartPanel2.add(chartz);
})

    chartPanel.add(chart);
    chartPanel.add(chartUHI);
    chartPanel.add(area_chart);
    chartPanel.add(chartMeanKec);
    chartPanel.add(chartMeanUHIKec);
    chartPanel.add(AreaChartUHI);
    //chartPanel.add(PopInfoLabel);
    chartPanel.add(popLabel);
    chartPanel.add(ChartTotalPop);
    chartmainPanel.add(chartPanel); 
    mainPanel.add(AddInfoLabel);
    mainPanel.add(koordPanel);
    mainPanel.add(inspectPanel);
    mainPanel.add(chartPanel2);
} //END OF FUNCTION
