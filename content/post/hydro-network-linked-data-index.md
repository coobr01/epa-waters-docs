+++
Categories = ["Data Science","Software Development","Applications"]
Description = ""
Tags = ["R","dataRetrieval","EGRET","geoknife","leaflet","Agile","Geo Data Portal"]
date = "2017-06-02T12:09:50-04:00"
menu = "main"
thumbnail = "/img/first_streamgage.jpeg"
title = "The Hydro Network-Linked Data Index"
author = "David Blodgett"
slug="nldi-intro"

+++

# Introduction

The Hydro [Network-Linked Data Index (NLDI)](https://cida.usgs.gov/nldi/about) is a system that can index data to [NHDPlus V2](http://www.horizon-systems.com/NHDPlus/V2NationalData.php) catchments and offers a search service to discover indexed information. Data linked to the NLDI includes [active NWIS stream gages](https://waterdata.usgs.gov/nwis/rt), [water quality portal sites](https://www.waterqualitydata.us/), and [outlets of HUC12 watersheds](https://www.sciencebase.gov/catalog/item/5762b664e4b07657d19a71ea). The NLDI is a core product of the [Open Water Data Initiative](http://acwi.gov/spatial/owdi/) and is being developed as an [open source project](https://github.com/ACWI-SSWD). coordinated through the [Advisory Committee on Water Information (ACWI) Subcommittee on Spatial Water Data](http://acwi.gov/spatial).

In this blog post, we introduce the basic functions of the NLDI and show how to use it as a data discovery and access tool in R. The first section describes the operations available from the [NLDI’s Web API](https://cida.usgs.gov/nldi/swagger-ui.html). The second section shows how to map NLDI data and how to use the NLDI to discover data to be accessed with the [dataRetrieval](https://github.com/USGS-R/dataRetrieval) R- package.

Below, text highlighting is used in six ways:

1. The names of API parameters such as `{featureSource}`
2. Example values of API parameters such as: `USGS-08279500`
3. API operation names: **_navigation_** and **_basin_**
4. API request names such as: getDataSources
5. R functions such as: **readOGR**
6. Other specific strings such as “siteNumber”

# The NLDI Web API

The NLDI’s Web API follows a loosely RESTful design and is documented with swagger documentation which can be found here. Every request to get data from the NLDI starts from a given network linked feature.

## Feature Sources

Available network linked feature sources ({featureSource}s) can be found from the getDataSources request. (hint: Click the “try it out” button on the swagger page!) These are the collections of network linked features the NLDI knows about. Think of them as watershed outlets that can be used as a starting point. For this demo, we’ll use NWIS Stream Gages as the {featureSource}. As a note for later, {featureSource} here is the same as the {dataSource} described below.

## Feature IDs

For now, the particular {featureID} to be accessed from a given {featureSource} needs to be found outside the NLDI, so the getFeatures request doesn’t return anything. So let’s use a well known NWIS Streamgage on the Rio Grande at Embudo NM as our starting point.

![Sample Image](sample1_l.jpg)

## Indexed Features

We can use the getRegisteredFeature request to see this feature. Enter nwissite and USGS-08279500 in the {featureSource} and {featureID}, respectively, in the swagger demo page. You can also see this in your browser at this url: https://cida.usgs.gov/nldi/nwissite/USGS-08279500 The response contains the location of the feature, is in geojson, and looks like:

```
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -105.9639722,
          36.20555556
        ]
      },
      "properties": {
        "source": "nwissite",
        "sourceName": "NWIS Sites",
        "identifier": "USGS-08279500",
        "name": "RIO GRANDE AT EMBUDO, NM",
        "uri": "https://waterdata.usgs.gov/nwis/inventory?agency_code=USGS&site_no=08279500",
        "comid": "17864756",
        "navigation": "https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate"
      }
    }
  ]
}
```

## Navigation

The navigation property of the returned feature is a url for the getNavigationTypes request. This request provides four navigation options as shown below. Each of these URLs returns the NHDPlus flowlines for the navigation type.

```
{
  "upstreamMain": "https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/UM",
  "upstreamTributaries": "https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/UT",
  "downstreamMain": "https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/DM",
  "downstreamDiversions": "https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/DD"
}
```

## Get Flowlines from Navigation

Each of the URLs found via the getNavigationTypes request is a complete getFlowlines request. This request has some optional input parameters. The most useful being {distance}, which allows specification of a distance to navigate in km. So, for example, we can use this to retrieve 150km of upstream mainstem flowlines from the NWIS gage 08279500 with a request like: https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/UM?distance=150

Notice that the flowline goes downstream of the gage because the NLDI is referenced to whole NHDPlus catchments, not to precise network locations.

## Get Linked Data from Navigation

Now that we have a {featureSource} = nwissite, a {featureID} = USGS-082795001, the navigate operation on the feature, and the {navigationMode} = UM with {distance} = 150km, we can use the getFeatures request to discover features from any {featureSource} which, in the context of a getFeatures request, is called a {dataSource}. Setting the {dataSource} = nwissite, we can see if there are any active NWIS streamgages 150km upstream on the main stem with a request that looks like: https://cida.usgs.gov/nldi/nwissite/USGS-08279500/navigate/UM/nwissite?distance=150 Note that we could enter wqp in place of nwissite after UM here to get water quality portal sites instead of NWIS sites. An example of this is shown later in this post.
