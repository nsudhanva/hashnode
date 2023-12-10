---
title: "CoGeoTIFF Research on Space Tech SaaS Platform at Pixxel"
datePublished: Sun Oct 10 2021 19:33:02 GMT+0000 (Coordinated Universal Time)
cuid: cl7amgopu02unaznv1c4j47su
slug: cogeotiff-research-on-space-tech-saas-platform-at-pixxel
canonical: https://sudhanva-narayana.ghost.io/cogeotiff-research-on-space-tech-saas-platform-at-pixxel/
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1661526339231/XO8r2pO1E.jpeg
tags: aws, space-tech, geotiff, co-geotiff

---

Introduction
------------

GeoTIFF is a public domain metadata standard which allows georeferencing information to be embedded within a TIFF file. The potential additional information includes map projection, coordinate systems and everything else necessary to establish the exact spatial reference for the file. While this is fine, these are large files. Serving large files over the internet (HTTP) is challenging. Even if it is decided to serve, it ends up costing more on the infrastructure side.

CoGeoTIFF
---------

The sole purpose of a co-geotiff [A Cloud Optimized GeoTIFF](https://www.cogeo.org/) is to help developers serve TIFF images over HTTP. It is a regular TIFF file that has been run through a compression. It does this by leveraging the ability of clients issuing ​HTTP GET range requests to ask for just the parts of a file they need.

### Creation of CoGeoTiff

The creation process is pretty straight forward. You can use [GDAL](https://gdal.org/) or [Rio CoGeo](https://github.com/cogeotiff/rio-cogeo) CLI tools to convert GeoTiff to CoGeoTiffs. Read about other challenges [here](https://medium.com/@_VincentS_/do-you-really-want-people-using-your-data-ec94cd94dc3f)

### Initial Development

The development of CoGeotiff was carried out by [CoGeotiff](https://github.com/cogeotiff/rio-cogeo) and [Vincent Sarago](https://github.com/cogeotiff/rio-cogeo/commits?author=vincentsarago). However, they only aid in the conversion of one image to another. Having a cogeotiff is not enough

### Rio Tiler

This tool helps you extract information out of a CoGeotiff. You can see documentation and examples [here](https://github.com/cogeotiff/rio-tiler)

Tile Server
-----------

A tile server is a server that serves geoinformation metadata. Usually returns JSON over HTTP. Initially, [Sentinel Tiler](https://github.com/mapbox/sentinel-tiler) was developed by Mapbox which is just a wrapper of RioTiler in a Flask application

### Marblecutter

Another example of a tile server: [Marble Cutter](https://github.com/mojodna/marblecutter-virtual). Initially developed to be deployed on AWS Lambdas.

### Remote Pixel Tiler

Another example of a tile server by Remote Pixel along with examples: [here](https://github.com/RemotePixel/remotepixel-tiler)

### CoGeo Tiler

Another example of a tile server from Vincent. Ready to be deployed to AWS Lamda. Find [here](https://github.com/developmentseed/cogeo-tiler)

### Titiler - Latest WIP

[Titiler](https://github.com/developmentseed/titiler/) is a lightweight service, which sole goal is to create map tiles dynamically from Cloud Optimized GeoTIFF. Also ready for deployment/use in production

Building A Product
------------------

While the above examples are tile servers, meaning they are still just servers which spits out JSON at best. A map layer such as OpenLayers or Leaflet.js or Geotiff.js (or a combination of these) have to be used to render maps onto a browser.

Example Products
----------------

Some of the examples that stale (not actively maintained) but do a good of explaining serving of cogeotiff along with a UI

*   [COG Map](https://cholmes.github.io/cog-map/)
*   [Titiler Templates](https://github.com/developmentseed/titiler/tree/master/titiler/templates) and [Notebooks](https://github.com/developmentseed/titiler/tree/master/Notebooks)
*   [Remote Pixel Tiler](https://github.com/RemotePixel/remotepixel-tiler) and [Remote Pixel Viewer](https://viewer.remotepixel.ca/)

Conversion from Sentinel 2 L1C to Sentinel 2 L2A
------------------------------------------------

### Case 1 - Tiff to COG conversion

Most of the raw data are available in tiff format. If it is decided to keep the original raw data, in the further processes such as rendering images on the client-side, these tiffs are to be converted to COG. This could lead to data replication (Warehousing both tiff and cog). Since tiff and cog have the same properties and cog doesn't lose any metadata associated with tiff files, COG would be the ideal solution

### Case 2 - COG to COG conversion

If the data is already in COG, which is again a compressed tiff with all properties of tiff, then further processes such as machine learning, or other processing will not be impacted. Hence, storing as COG will be an advantage both in terms of storage space and further processing

### Storage, Networking, Processing

Sentinel-2 scenes hosted on AWS are not in Cloud Optimized format but in JPEG2000. When performing a partial reading of the JPEG2000 dataset GDAL (rasterio backend library) will need to make a lot of GET requests and transfer a lot of data. This problem can only be solved by asking the user to select the boundary required for download, then convert those JP2 to COG. A region/tile previously requested/converted by a user could be logged and need not be converted again.

*   Worst case scenario is ending up converting everything to COG (It is mentioned [here](https://gist.github.com/vincentsarago/0e8c3fba19a4b2b6855bca77f18b88fb) that if every tile is converted, it should cost around <$100,000 hypothetically)
*   Best case scenario is converting **only** the required tiles and then avoiding re-conversion of tiles/images

**Downloading from S3**: A researcher decides to download 1 tile of Sentinel 1 and Sentinel 2 L1 and L2 each. Assuming the tile size of S1 being 7.5GB and S2 being L1 and L2 each 4.5GB, a total download size of 16.5 GB. A pipeline on Glue created to extract these files, find fourteen bandwidths of images, and transform all of them to CoGeoTiff or any other format. Each conversion takes 10 seconds (per image). This Glue pipeline is then triggered by Lambda. Converted files are again stored in AWS S3.  
Quota - 16.5GB internet download bandwidth ~= 7 \* 16.5 = ₹289  
Transformation assuming each image is 1GB in size - 10 seconds \* 14 images \* 3 satellites = 420 seconds ~= 7 minutes ~= ₹35  
Lamda - assuming after free quota - ₹15  
Total Pricing = 289 + 35 + 15 = ₹340

**Downloading from Copernicus**: A developer decided to download one hundred tiles of Sentinel-1 images. Assuming each tile being 7.5GB on average since Copernicus limits downloading to two downloads per user, we must use a queue. Download time takes around 10 minutes per time. Assuming the free quota is over, the total size would be 7.5GB \* 100 = 750GB and 350 queues. A pipeline on Glue created to extract these files, find fourteen bandwidth of images, and transform all of them to CoGeoTiff or any other format. Each conversion takes 10 seconds (per image). This Glue pipeline is then triggered by Lambda. Converted files are again stored in AWS S3.  
Quota - 750GB internet bandwidth - Copernicus egress and AWS Ingress is free  
Transformation assuming each image is 1GB in size - 10 seconds \* 14 images \* 100 tiles \* 1 satellite = 140 seconds \* 100 ~= 3 hr ~= ₹105  
SQS for 350 queues ~= ₹30  
Total Pricing = 105 + 30 = ₹135

Either way, the output format should be in COG because it aligns well with the rest of the pipeline

Sentinel L2A to ML Case
-----------------------

### Case 1 - Client-Side Rendering of COG

The key to making this work well is coordinating the data delivery between the client and server, through COGs, which can deliver just the information needed for the current view. More work is needed for this to operate seamlessly, but most JS stacks run the same javascript code in the browser and the server for maximum flexibility

### Case 2 - COG to COG conversion

The ML model is unaffected by the input. Be it Tiff or Cog. Since they are virtually the same, the training process, time of convergence would remain the same.

> Doing everything on-demand and through an API then opens many possibilities to tailor the data more to how the end-user wants it. The first of these was to enable ‘clipping’, which lets a user request just the geometry they care about, instead of trying to select the scenes that overlap with their area of interest. This will evolve to enable co-registration of images, application of TOA, atmospheric correction, surface reflectance, and eventually full analytic processing of images with operations like band math to create indices and even computer vision-based object detection.

### Storage, Networking, Processing

Storage cost of JPEG2000 is less compared to that of COGs, but someone will have to pay to access/process the data. If the data is being stored and used for providing services around, COG should be a better long-term solution.

> Short answer to the question: there is no such thing as an Ultimate data format, in the real world there are plenty of good data formats. At the end of the day, it relies on what you want the user to do.

#### COG Client-Side Rendering

Size: 50 Tb.

*   Storage: 50 Tb \* 1000 \* 0.023 = 86,000 Rs / month\*
*   Data access: (1M \* 5 (GET requests) / 1000) \* 0.004 = Rs. 1500\*
*   Processing time (1GB AWS Lambda): (1 second \* 1M \* 1GB / 1000) \* 0.00001667 $ = Rs 2000\*

_Reading a tile from a COG is at least 3-4 times faster than for JPEG2000_

**Cost:** 86k + 1500 + 2000 = Rs. 90,000 (\*Estimated +/- 10k for processing, network transfer)

Future
------

COG is incredibly powerful and there is an emerging ecosystem of geospatial algorithms that can run fully client-side. It leverages a newer feature of HTTP called Byte Serving. [Hotosm's ML Enabler](https://github.com/hotosm/ml-enabler) allows you to apply ML algorithms to maps directly on the browser. [Here](https://medium.com/devseed/ml-enabler-completing-the-machine-learning-pipeline-for-mapping-3aae94fa9e94) it is explained how it makes it easier to collect and organize AI-derived map information. Soon, the task of [applying ML](https://medium.com/devseed/mapping-buildings-with-help-from-machine-learning-f8d8d221214a) to maps is going to be reduced. The project is not open source yet, but they plan to release it as the work moves from a research project to a more stable tool.

Conclusion
----------

Building a custom tile server requires a good understanding of OpenLayers or Leaflet or GeoTiff.js along with the ability to request information (React.js state like instant updates) from the server and render the same on the client-side. While most of the images are stored in AWS S3 buckets, they can be served right from there. However, some of them are not stored as tiff. Those could pose a potential increase in conversion and storage costs.

Checkout: [https://pixxel.space](https://pixxel.space/)

Footnotes
---------

### Theory behind COG

*   [https://medium.com/@vrielink/whats-wrong-with-open-infrastructure-for-remote-sensing-geodata-af55c91e0f03](https://medium.com/@vrielink/whats-wrong-with-open-infrastructure-for-remote-sensing-geodata-af55c91e0f03)
*   [https://medium.com/@_VincentS_/do-you-really-want-people-using-your-data-ec94cd94dc3f](https://medium.com/@_VincentS_/do-you-really-want-people-using-your-data-ec94cd94dc3f)
*   [https://blog.maxar.com/earth-intelligence/2018/cloud-optimized-geotiffs-and-the-path-to-accessible-satellite-imagery-analytics](https://blog.maxar.com/earth-intelligence/2018/cloud-optimized-geotiffs-and-the-path-to-accessible-satellite-imagery-analytics)
*   [https://medium.com/planet-stories/cloud-native-geospatial-part-1-basic-assumptions-and-workflows-aa67b6156b53](https://medium.com/planet-stories/cloud-native-geospatial-part-1-basic-assumptions-and-workflows-aa67b6156b53)
*   [https://medium.com/planet-stories/cloud-native-geospatial-part-2-the-cloud-optimized-geotiff-6b3f15c696ed](https://medium.com/planet-stories/cloud-native-geospatial-part-2-the-cloud-optimized-geotiff-6b3f15c696ed)
*   [https://medium.com/radiant-earth-insights/cloud-optimized-geotiff-advances-6b01750eb5ac](https://medium.com/radiant-earth-insights/cloud-optimized-geotiff-advances-6b01750eb5ac)

### Cloud Native Architechture

*   [https://medium.com/planet-stories/cloud-native-geoprocessing-part-1-the-basics-9670280772c8](https://medium.com/planet-stories/cloud-native-geoprocessing-part-1-the-basics-9670280772c8)
*   [https://medium.com/planet-stories/cloud-native-geospatial-part-2-the-cloud-optimized-geotiff-6b3f15c696ed](https://medium.com/planet-stories/cloud-native-geospatial-part-2-the-cloud-optimized-geotiff-6b3f15c696ed)
*   [https://medium.com/planet-stories/cng-part-3-planets-cloud-native-geospatial-architecture-31fb4a20fa77](https://medium.com/planet-stories/cng-part-3-planets-cloud-native-geospatial-architecture-31fb4a20fa77)
*   [https://medium.com/planet-stories/cng-part-4-open-aerial-maps-cloud-native-geospatial-architecture-a7f784cf7c2f](https://medium.com/planet-stories/cng-part-4-open-aerial-maps-cloud-native-geospatial-architecture-a7f784cf7c2f)
*   [https://medium.com/planet-stories/cng-part-5-cloud-native-geospatial-architecture-defined-193d5ffdd681](https://medium.com/planet-stories/cng-part-5-cloud-native-geospatial-architecture-defined-193d5ffdd681)
*   [https://medium.com/planet-stories/cng-part-6-metadata-in-a-cloud-native-geospatial-world-2fa83cc00c95](https://medium.com/planet-stories/cng-part-6-metadata-in-a-cloud-native-geospatial-world-2fa83cc00c95)
*   [https://medium.com/planet-stories/analysis-ready-data-defined-5694f6f48815](https://medium.com/planet-stories/analysis-ready-data-defined-5694f6f48815)
*   [https://medium.com/planet-stories/towards-on-demand-analysis-ready-data-f94d6eb226fc](https://medium.com/planet-stories/towards-on-demand-analysis-ready-data-f94d6eb226fc)
*   [https://github.com/mojodna/marblecutter-virtual](https://github.com/mojodna/marblecutter-virtual)
*   [https://github.com/mojodna](https://github.com/mojodna)
*   [https://github.com/cogeotiff/rio-cogeo](https://github.com/cogeotiff/rio-cogeo)
*   [https://github.com/cholmes/cog-map/](https://github.com/cholmes/cog-map/)
*   [https://github.com/cogeotiff/rio-tiler-mosaic](https://github.com/cogeotiff/rio-tiler-mosaic)
*   [https://github.com/vincentsarago/lambda-tiler](https://github.com/vincentsarago/lambda-tiler)
*   [https://github.com/vincentsarago](https://github.com/vincentsarago)
*   [https://github.com/cogeotiff/rio-tiler](https://github.com/cogeotiff/rio-tiler)
*   [https://geotiffjs.github.io/](https://geotiffjs.github.io/)
*   [https://github.com/radiantearth/stac-spec](https://github.com/radiantearth/stac-spec)
*   [https://medium.com/@cholmes/more-cloud-optimized-geotiff-qgis-tutorial-and-an-online-cog-validator-f4115c9cbe14](https://medium.com/@cholmes/more-cloud-optimized-geotiff-qgis-tutorial-and-an-online-cog-validator-f4115c9cbe14)

[Previous issue](https://sudhanva-narayana.ghost.io/leaving-pixxel/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/analyzing-data-for-a-food-tech-startup-swiggy/)