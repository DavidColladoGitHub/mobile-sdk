# Custom Offline data

## Types

CartoMobileSDK supports several different types of custom sources: 

* mbtiles
* geojson
* Map preloader

In addition to custom sources, your data can be stored on CARTO servers and accessed via

* tile downloader
* map preloader

### MBTiles

MBTiles is a file format for storing map tiles in a single file. It is, technically, a SQLite database. Latest format version as of 21 October 2011 is 1.1, next version (1.2) is in development. [wiki.openstreetmap.org](http://wiki.openstreetmap.org/wiki/MBTiles)

The following is an example of how you can load mbiles from bundled assets and render it on the map:

<div class="js-TabPanes">
  <ul class="Tabs">
    <li class="Tab js-Tabpanes-navItem--lang is-active">
      <a href="#/0" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--java">Java</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/1" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--csharp">C#</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/2" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--objective-c">Objective-C</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/3" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--swift">Swift</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/3" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--kotlin">Kotlin</a>
    </li>
  </ul>

   <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--java is-active">
  {% highlight java %}

// Copy this method to your class
protected TileDataSource createTileDataSource() {

    // Offline map data source
    String mbTileFile = "rome_ntvt.mbtiles";

    try {
        String localDir = getExternalFilesDir(null).toString();
        copyAssetToSDCard(getAssets(), mbTileFile, localDir);

        String path = localDir + "/" + mbTileFile;
        Log.i(MapApplication.LOG_TAG,"copy done to " + path);
        MBTilesTileDataSource vectorTileDataSource = new MBTilesTileDataSource(0, 19, path);

        return vectorTileDataSource;

    } catch (IOException e) {
        Log.e(MapApplication.LOG_TAG, "mbTileFile cannot be copied: " + mbTileFile);
        Log.e(MapApplication.LOG_TAG, e.getLocalizedMessage());
    }

	return null;
}

// This one as well
public void copyAssetToSDCard(AssetManager assetManager, String fileName, String toDir) throws IOException {

    InputStream in = assetManager.open(fileName);
    File outFile = new File(toDir, fileName);

    // NB! Remember to check if storage is available and has enough space

    if (outFile.exists()) {
        // File already exists, no need to recreate
        return;
    }

    OutputStream out = new FileOutputStream(outFile);
    copyFile(in, out);
    in.close();
    in = null;
    out.flush();
    out.close();
    out = null;
}

// Then initialize your layer as such:
addBaseLayer(CartoBaseMapStyle.CARTO_BASEMAP_STYLE_VOYAGER);

TileDataSource source = createTileDataSource();

// Get decoder from current layer,
// so we wouldn't need a style asset to create a decoder from scratch
MBVectorTileDecoder decoder = (MBVectorTileDecoder)((VectorTileLayer)mapView.getLayers().get(0)).getTileDecoder();

// Remove default baselayer
mapView.getLayers().clear();

// Add our new layer
VectorTileLayer layer = new VectorTileLayer(source, decoder);
mapView.getLayers().insert(0, layer);

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--csharp">
  {% highlight csharp %}
  
// Copy this method to your class
TileDataSource CreateTileDataSource()
{
	// offline map data source
	string fileName = "<bundled-asset-name>";

	try
	{
		string directory = GetExternalFilesDir(null).ToString();
		string path = directory + "/" + fileName;

		Assets.CopyAssetToSDCard(fileName, path);
		Log.Debug("Copy done to " + path);

		MBTilesTileDataSource source = new MBTilesTileDataSource(0, 14, path);

		return new MemoryCacheTileDataSource(source);
	}
	catch (IOException e)
	{
		Log.Debug("MbTileFile cannot be copied: " + fileName);
		Log.Debug("Message" + e.LocalizedMessage);
	}

	return null;
}

// And then you can simply call it as such:
			AddOnlineBaseLayer(CartoBaseMapStyle.CartoBasemapStyleDefault);

// Get decoder from current layer,
// so we wouldn't need a style asset to create a decoder from scratch
MBVectorTileDecoder decoder = (MBVectorTileDecoder)(MapView.Layers[0] as VectorTileLayer).TileDecoder;

// Remove default baselayer
MapView.Layers.Clear();

// Do the actual copying and source creation on another thread so it wouldn't block the main thread
System.Threading.Tasks.Task.Run(delegate
{
	TileDataSource source = CreateTileDataSource();

	var layer = new VectorTileLayer(source, decoder);

	// However, actual layer insertion should be done on the main thread
	RunOnUiThread(delegate
	{
		MapView.Layers.Insert(0, layer);
	});
});
			
  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--objective-c">
  {% highlight objc %}

// Copy this function to your class
- (NTTileDataSource*)createTileDataSource
{
    NSString *name = @"<your-file-name>";
    NSString *extension = @"mbtiles";
    
    // file-based local offline datasource
    NSString* source = [[NSBundle mainBundle] pathForResource:name ofType:extension];
    
    NTTileDataSource* vectorTileDataSource = [[NTMBTilesTileDataSource alloc] initWithMinZoom:0 maxZoom:14 path:source];
    return vectorTileDataSource;
}

// And then call it and create the layer as such:

// Get the base projection set in the base class
NTProjection* projection = [[self.mapView getOptions] getBaseProjection];
NTTileDataSource *source= [self createTileDataSource];

NTCartoOnlineVectorTileLayer *baseLayer = [[NTCartoOnlineVectorTileLayer alloc]initWithStyle:NT_CARTO_BASEMAP_STYLE_VOYAGER];
NTVectorTileDecoder *decoder = [baseLayer getTileDecoder];
    
NTVectorTileLayer *layer = [[NTVectorTileLayer alloc]initWithDataSource:source decoder:decoder];
[[self.mapView getLayers] add:layer];
    
  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--swift">
  {% highlight swift %}
  
let baseLayer = NTCartoOnlineVectorTileLayer(style: .CARTO_BASEMAP_STYLE_VOYAGER);
let decoder = baseLayer?.getTileDecoder()
    
// Do the actual copying and source creation on another thread so it wouldn't block the main thread
DispatchQueue.global().async {
    
    let path = Bundle.main.path(forResource: "<your-file-name", ofType: "mbtiles")
    let source = NTMBTilesTileDataSource(minZoom: 0, maxZoom: 14, path: path)
    let layer = NTVectorTileLayer(dataSource: source, decoder: decoder)
    
    DispatchQueue.main.async {
        // But add the layer on the main thread
        self.contentView.map.getLayers().add(layer)
    }
}

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--kotlin">
  {% highlight kotlin %}
  
val baseLayer = CartoOnlineVectorTileLayer(CartoBaseMapStyle.CARTO_BASEMAP_STYLE_VOYAGER)
val decoder = baseLayer.tileDecoder

// Do the actual copying and source creation on another thread so it wouldn't block the main thread
// This requires the anko coroutines library:
// compile "org.jetbrains.anko:anko-sdk25-coroutines:$anko_version"
doAsync {

    val filename = "<your-file-name>.mbtiles"
    val directory = getExternalFilesDir(null).toString()
    val path = directory + "/" + filename

    // Copy assets to SD card
    val input = assets.open(filename)
    val outFile = File(filename, directory)

    if (!outFile.exists()) {
        val output = FileOutputStream(outFile)
        input.copyTo(output)
        input.close()
        output.close()
    }

    val source = MBTilesTileDataSource(0, 14, path)
    val layer = VectorTileLayer(source, decoder)
    
    runOnUiThread {
        contentView!!.map.layers.add(layer)
    }
}

  {% endhighlight %}
  </div>
</div>

### GeoJson

GeoJSON is a format for encoding a variety of geographic data structures. GeoJSON supports the following geometry types: Point, LineString, Polygon, MultiPoint, MultiLineString, and MultiPolygon. Geometric objects with additional properties are Feature objects. Sets of features are contained by FeatureCollection objects. [geojson.org](http://geojson.org/)

The following is an example of how you can load geojson from bundled assets and render it on the map (make sure you have cities15000.geojson as a bundled Asset (Android) or Resource (iOS):

<div class="js-TabPanes">
  <ul class="Tabs">
    <li class="Tab js-Tabpanes-navItem--lang is-active">
      <a href="#/0" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--java">Java</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/1" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--csharp">C#</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/2" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--objective-c">Objective-C</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/3" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--swift">Swift</a>
    </li>
    <li class="Tab js-Tabpanes-navItem--lang">
      <a href="#/3" class="js-Tabpanes-navLink--lang js-Tabpanes-navLink--lang--kotlin">Kotlin</a>
    </li>
  </ul>

   <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--java is-active">
  {% highlight java %}

// Initialize a local vector data source
final Projection projection = mapView.getOptions().getBaseProjection();
final LocalVectorDataSource source = new LocalVectorDataSource(projection);
VectorLayer layer = new VectorLayer(source);
mapView.getLayers().add(layer);

// As the file to load is rather large, we don't want to block our main thread
thread = new Thread(new Runnable() {
    @Override
    public void run() {

        // Create a basic style
        MarkerStyle style = new MarkerStyleBuilder().buildStyle();

        // Read GeoJSON, parse it using SDK GeoJSON parser
        GeoJSONGeometryReader reader = new GeoJSONGeometryReader();

        // Set target projection to base (mercator)
        reader.setTargetProjection(projection);

        String fileName = "cities15000.geojson";
        String json;

        try {
            InputStream is = getAssets().open(fileName);
            int size = is.available();
            byte[] buffer = new byte[size];
            is.read(buffer);
            is.close();

            json = new String(buffer, "UTF-8");

        } catch (IOException ex) {
            return;
        }

        // Read features from local asset
        FeatureCollection features = reader.readFeatureCollection(json);

        VectorElementVector elements = new VectorElementVector();

        for (int i = 0; i < features.getFeatureCount(); i++) {
            // This data set features point geometry,
            // however, it can also be LineGeometry or PolygonGeometry
            PointGeometry geometry = (PointGeometry) features.getFeature(i).getGeometry();
            elements.add(new Marker(geometry, style));
        }

        source.addAll(elements);
    }
});

thread.start();

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--csharp">
  {% highlight csharp %}
	
// Read json from assets and add to map
string json;

using (System.IO.StreamReader sr = new System.IO.StreamReader (Assets.Open ("cities15000.geojson")))
{
	json = sr.ReadToEnd ();
}

// Initialize a local vector data source
var projection = MapView.Options.BaseProjection;
var source = new LocalVectorDataSource(projection);

VectorLayer layer = new VectorLayer(source);

new System.Threading.Thread((obj) =>
{
	// Create basic style. Use the builder to modify your markers
	MarkerStyle style = new MarkerStyleBuilder().BuildStyle();

	// Read GeoJSON, parse it using SDK GeoJSON parser
	GeoJSONGeometryReader reader = new GeoJSONGeometryReader();

	// Set target projection to base (mercator)
	reader.TargetProjection = projection;
	
	// Read features from local asset
	FeatureCollection features = reader.ReadFeatureCollection(json);
	
	VectorElementVector elements = new VectorElementVector();

	for (int i = 0; i < features.FeatureCount; i++)
	{
		// This data set features point geometry,
		// however, it can also be LineGeometry or PolygonGeometry
		PointGeometry geometry = (PointGeometry)features.GetFeature(i).Geometry;
		elements.Add(new Marker(geometry, style));
	}

	source.AddAll(elements);
	
	// Add the clustered vector layer to the map
	MapView.Layers.Add(layer);

}).Start();

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--objective-c">
  {% highlight objc %}

// Initialize a local vector data source
NTProjection *projection = [[self.mapView getOptions] getBaseProjection];
NTLocalVectorDataSource *source = [[NTLocalVectorDataSource alloc] initWithProjection:projection];
NTVectorLayer *layer = [[NTVectorLayer alloc]initWithDataSource:source];
[[[self mapView] getLayers] add:layer];
    
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0ul);
dispatch_async(queue, ^{
    // Read .geojson
    NSString* fullpath = [[NSBundle mainBundle] pathForResource:@"cities15000" ofType:@"geojson"];
    NSString* json = [NSString stringWithContentsOfFile:fullpath encoding:NSUTF8StringEncoding error:nil];
    
    // .geojson parsing
    NTGeoJSONGeometryReader* geoJsonReader = [[NTGeoJSONGeometryReader alloc] init];
    [geoJsonReader setTargetProjection:projection];
    
    NTFeatureCollection* features = [geoJsonReader readFeatureCollection:json];
    
    // Initialize basic style, as it will later be overridden anyway
    NTMarkerStyle *style = [[[NTMarkerStyleBuilder alloc] init] buildStyle];
    
    NTVectorElementVector *elements = [[NTVectorElementVector alloc]init];
    
    for (int i = 0; i < [features getFeatureCount]; i++) {
        NTPointGeometry *geometry = (NTPointGeometry *)[[features getFeature:i] getGeometry];
    
        NTMarker *marker = [[NTMarker alloc] initWithGeometry:geometry style:style];
        [elements add:marker];
    }
    
    // Add them all at once to avoid flickering
    [source addAll:elements];
});

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--swift">
  {% highlight swift %}

let projection = contentView.map.getOptions().getBaseProjection()
let source = NTLocalVectorDataSource(projection: projection)
let layer = NTVectorLayer(dataSource: source)
    
contentView.map.getLayers().add(layer)
    
DispatchQueue.global().async {

    let path = Bundle.main.path(forResource: "cities15000", ofType: "geojson")
    
    guard let json = try? NSString(contentsOfFile: path!, encoding: String.Encoding.utf8.rawValue) else {
        return
    }
    
    // Create default style
    let mBuilder = NTMarkerStyleBuilder()
    let style = mBuilder?.buildStyle()
    
    // Read GeoJSON, parse it using SDK GeoJSON parser
    let reader = NTGeoJSONGeometryReader()
    reader?.setTargetProjection(self.contentView.map.getOptions().getBaseProjection())
    
    let features = reader?.readFeatureCollection(json as String!)
    
    let elements = NTVectorElementVector()
    let total = Int((features?.getFeatureCount())!)
    
    
    for i in stride(from: 0, to: total, by: 1) {
        // This data set features point geometry, however, it can also be LineGeometry or PolygonGeometry
        let geometry = features?.getFeature(Int32(i)).getGeometry() as? NTPointGeometry
        elements?.add(NTMarker(geometry: geometry, style: style))
    }
    
    DispatchQueue.main.async(execute: {
        source?.addAll(elements)
    })
}

  {% endhighlight %}
  </div>

  <div class="Carousel-item js-Tabpanes-item--lang js-Tabpanes-item--lang--kotlin">
  {% highlight kotlin %}
  
  
val source = LocalVectorDataSource(contentView!!.map.options.baseProjection)
val layer = VectorLayer(source)
contentView!!.map.layers.add(layer)

// Kotlin Anko library: https://github.com/Kotlin/anko
doAsync {

    val filename = "cities15000.geojson"

    val stream = assets.open(filename)
    val size = stream.available()
    val buffer = ByteArray(size)

    stream.read(buffer)
    stream.close()

    val json = String(buffer, charset("UTF-8"))

    // Basic style builder
    val mBuilder = MarkerStyleBuilder()
    val style = mBuilder.buildStyle()

    // Read GeoJSON, parse it using SDK GeoJSON parser
    val reader = GeoJSONGeometryReader()
    reader.targetProjection = contentView?.projection

    val features = reader.readFeatureCollection(json)

    val elements = VectorElementVector()
    val total = features.featureCount

    for(i in 0..total - 1) {
        // This data set features point geometry, however, it can also be LineGeometry or PolygonGeometry
        val geometry = features.getFeature(i).geometry as PointGeometry
        elements.add(Marker(geometry, style))
    }

    source.addAll(elements)
}

  {% endhighlight %}
  </div>
</div>


