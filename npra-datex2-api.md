# NPRA DATEX II 3.1 API — Documentation

## Overview

The Norwegian Public Roads Administration (NPRA / Statens vegvesen) publishes real-time road and
traffic data via the DATEX II 3.1 standard. All data is free to use after registration, and is
licensed under the [Norwegian License for Open Government Data (NLOD 2.0)](https://data.norge.no/nlod/en/2.0).

**Base URL:** `https://datex-server-get-v3-1.atlas.vegvesen.no/`

**Registration:** https://www.vegvesen.no/en/fag/technology/open-data/a-selection-of-open-data/what-is-datex/get-access/

---

## Attribution

When using this data, acknowledge NPRA as the source. Example:

> Made with data from [The Norwegian Public Roads Administration](https://www.vegvesen.no/en/).

---

## Authentication

All services are secured with **Basic Authentication** (username/password issued upon registration).

---

## Technical Details

- **Delivery mode:** PULL only (no push/streaming)
- **Protocols:** SOAP and HTTP GET
- **Format:** XML
- **Caching support:** `If-Modified-Since` / `Last-Modified` headers supported
  - Send `If-Modified-Since` with the value from a previous `Last-Modified` response
  - If no new data exists, server returns `304 Not Modified` — saves bandwidth on polling

---

## Data Feeds

### 1. Travel Times
> Updated every **5 minutes**

| Name | Description | HTTP GET |
|------|-------------|----------|
| PredefinedTravelTimeLocations | Geometry/location of road segments | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetPredefinedTravelTimeLocations/pullsnapshotdata) |
| TravelTimeData | Travel times in seconds between measurement points | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetTravelTimeData/pullsnapshotdata) |

---

### 2. Weather Observations
> Updated every **10 minutes**

Weather measurements from permanent stations along national and county roads.

| Name | Description | HTTP GET |
|------|-------------|----------|
| MeasuredWeatherData | Weather observations from roadside stations | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetMeasuredWeatherData/pullsnapshotdata) |
| MeasurementSiteTable | Locations of weather stations | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetMeasurementWeatherSiteTable/pullsnapshotdata) |

---

### 3. Weather Forecasts
> Updated every **hour**, 24 hours ahead

| Name | Description | HTTP GET |
|------|-------------|----------|
| ForecastPointData | 24h weather forecasts at fixed road locations | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetForecastPointData/pullsnapshotdata) |
| ForecastPointLocations | Locations for weather forecast points | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetForecastPointLocations/pullsnapshotdata) |

---

### 4. Webcameras

Images from roadside cameras showing traffic flow, weather, and road surface conditions.

| Name | Description | HTTP GET |
|------|-------------|----------|
| CcTvSiteTable | Camera locations with links to images/video | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetCCTVSiteTable/pullsnapshotdata) |
| CcTvStatus | Camera status (active/inactive) | [pullsnapshotdata](https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetCCTVStatus/pullsnapshotdata) |

---

### 5. Road Traffic Information (Situations)

Incidents and events that may cause delays or increased accident risk.

**Base endpoint:**
`https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetSituation/pullsnapshotdata`

**Available filters** (append `/filter/<FilterName>` to the URL):

| Filter | Description |
|--------|-------------|
| Accident | Accidents |
| MaintenanceWorks | Road maintenance |
| ConstructionWorks | Construction |
| RoadOrCarriagewayOrLaneManagement | Lane/road management |
| WeatherRelatedRoadConditions | Weather-caused road conditions |
| WinterDrivingManagement | Winter driving measures |
| VehicleObstruction | Vehicles blocking road |
| GeneralObstruction | Other obstructions |
| AbnormalTraffic | Unusual traffic conditions |
| SpeedManagement | Speed restrictions |
| NetworkManagement | Network-level management |
| PoorEnvironmentConditions | Visibility, pollution etc. |
| EnvironmentalObstruction | Fallen trees, flooding etc. |
| AnimalPresenceObstruction | Animals on road |
| ReroutingManagement | Detour instructions |
| PublicEvent | Events affecting traffic |

**Example filter URLs:**

```
# Accidents only
https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetSituation/pullsnapshotdata/filter/Accident

# Maintenance works only
https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetSituation/pullsnapshotdata/filter/MaintenanceWorks

# Safety-Related Traffic Information (SRTI) only
https://datex-server-get-v3-1.atlas.vegvesen.no/datexapi/GetSituation/pullsnapshotdata?srti=True
```

---

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 OK | Successful request |
| 200 OK + "Delivery break" | Data may be outdated — source has not sent new data recently |
| 304 Not Modified | No new data since `If-Modified-Since` timestamp |
| 400 Bad Request | Invalid filter or parameter |
| 401 Unauthorized | Invalid credentials |
| 403 Forbidden | Non-existent endpoint or missing access rights |
| 404 Not Found | No entry found for given ID |
| 500 Internal Server Error | Server-side error |

---

## Supplementary Data: Traffic Registration Points

In addition to DATEX II, NPRA also provides traffic volume data via a **GraphQL API**:

**API endpoint:** `https://trafikkdata-api.atlas.vegvesen.no/`

Traffic registration points are physical sensor locations (inductive loops etc.) embedded in the road,
each with a unique ID, geographic coordinates, and metadata (road name, direction, lane).

### Example: Hourly Traffic Volume

```graphql
{
  trafficData(trafficRegistrationPointId: "44656V72812") {
    volume {
      byHour(
        from: "2019-10-24T12:00:00+02:00"
        to: "2019-10-24T14:00:00+02:00"
      ) {
        edges {
          node {
            from
            to
            total {
              volumeNumbers {
                volume
              }
              coverage {
                percentage
              }
            }
          }
        }
      }
    }
  }
}
```

### Example: List Registration Points with Coordinates

```graphql
{
  trafficRegistrationPoints {
    id
    name
    location {
      coordinates {
        latLon {
          lat
          lon
        }
      }
    }
  }
}
```

---

## Relevance for Navigation (e.g. Navit)

| Data | Use Case |
|------|----------|
| WeatherRelatedRoadConditions | Warn driver of icy/flooded roads ahead |
| WinterDrivingManagement | Alert to winter tyre requirements or chain requirements |
| MeasuredWeatherData | Adjust ETA based on current road weather |
| ForecastPointData | Proactive rerouting based on upcoming conditions |
| TravelTimeData | More accurate ETA calculations |
| Situations (Accident, Roadwork) | Real-time rerouting around incidents |
| CCTV | Visual road condition verification |

---

---

## Other European DATEX II Providers

Several other European countries publish traffic data in DATEX II format, many free of charge after registration. Below is an overview of known providers.

---

### Sweden — Trafikverket (Swedish Transport Administration)

- **Portal:** https://data.trafikverket.se/documentation/datex
- **Info:** https://bransch.trafikverket.se/en/startpage/operations/Operations-road/Traffic-information/Real-time-traffic-information/
- **Registration:** Required, free of charge. Agreement must be signed before access is granted.
- **License:** Custom license with few restrictions; commercial use appears permitted.
- **Delivery modes:** Push (updates sent as soon as they arrive) and Pull
- **Format:** XML (DATEX II), also JSON via separate open API
- **DATEX II version:** 3.0+
- **Data available:**
  - Road traffic incidents and situations
  - Roadworks and temporary obstacles
  - Traffic safety information (SRTI)
  - Road weather and reduced visibility
  - Traffic flow and travel times
  - Road geometry and static road network data
- **Open API (no registration):** `https://api.trafikinfo.trafikverket.se/` — REST/JSON API for road and rail traffic data, requires only an API key (obtainable via [Trafiklab](https://www.trafiklab.se/))

---

### Finland — Digitraffic / Fintraffic

- **Portal:** https://www.digitraffic.fi/en/road-traffic/
- **Registration:** None required — fully open
- **License:** Creative Commons 4.0 BY
- **Format:** DATEX II (XML and JSON), also GeoJSON
- **DATEX II version:** 3.5
- **Base URL:** `https://tie.digitraffic.fi/`
- **Data available:**
  - Road weather camera images (470+ cameras)
  - Road weather station measurements — temperature, wind, rain, humidity, dew point (updated every minute, 350+ stations)
  - Road weather forecasts (updated every 5 minutes)
  - Traffic Measurement System (TMS) data — average speeds and traffic volumes in DATEX II format
  - Traffic messages — incidents, weight restrictions, roadworks in DATEX II and GeoJSON

**Example DATEX II endpoints:**

| Data | JSON | XML |
|------|------|-----|
| TMS station metadata | `/api/tms/v1/stations/datex2` | `/api/tms/v1/stations/datex2.xml` |
| TMS station data | `/api/tms/v1/stations/data/datex2` | `/api/tms/v1/stations/data/datex2.xml` |
| Traffic messages | `/api/traffic-message/v1/messages` | `/api/traffic-message/v1/messages.xml` |

---

### Netherlands — NDW (Nationale Databank Wegverkeersgegevens)

- **Open data portal:** https://opendata.ndw.nu/
- **Documentation:** https://docs.ndw.nu/en/
- **Registration:** Not required for open data feeds
- **Format:** DATEX II (XML, gzipped), also JSON REST API
- **DATEX II version:** Migrating from v2.3 to v3 (migration deadline April 2, 2026)
- **Data available:**
  - Traffic flow, speed and travel times from 24,000+ measurement locations (updated every minute)
  - Traffic incidents — jams, wrong-way drivers, closures, detours, weather conditions
  - Roadworks and event-related traffic measures
  - Bridge opening timetables (movable bridges)
  - Traffic signs
  - Vehicle categories (3-category and 5-category classification)
- **Open data feeds (no auth required):**
  - `http://opendata.ndw.nu/` — index of all feeds (gzipped XML)
  - `http://opendata.ndw.nu/wegwerkzaamheden.xml.gz` — roadworks
  - `http://opendata.ndw.nu/gebeurtenisinfo.xml.gz` — traffic events/incidents
- **Note:** NDW uses a custom location code list (VILD) instead of standard Alert-C. Download from https://www.ndw.nu/documenten/nl/#cat_2

---

### Other Known Providers (summary)

| Country | Provider | Registration | Format | Notes |
|---------|----------|-------------|--------|-------|
| Belgium | Flemish/Walloon regional authorities | Free | DATEX II | Roadworks; uses Alert-C (use default table, ignore stream LTN) |
| Austria | ASFINAG / regional | Required; some free, some paid | DATEX II | Various datasets; also an open RSS feed |
| Czechia | ŘSD / TMC | Required, free | DATEX II + DDR XML | Push-based; requires running a server to receive data |
| Estonia | Maanteeamet | Required, free | DATEX II | Road safety, restrictions, real-time flow; WGS84 coordinates |
| Luxembourg | Administration des ponts et chaussées | Free | DATEX II | Traffic events; uses Alert-C |
| Germany | Mobilithek (replaces MDM/mCLOUD) | Varies | DATEX II + others | National platform aggregating data from multiple Länder |
| France | CEREMA / DIRXX | Varies | DATEX II | Traffic counts and event data; Paris has separate counter feeds |

---

### Location Referencing Note

Most European DATEX II feeds use **Alert-C (TMC)** for location encoding. To decode locations you need the country's **Location Code List (LCL)**:
- LCLs are generally free to obtain
- A FOSS Java library for Alert-C decoding: [traff-libalertclocation](https://github.com/bafto/traff-libalertclocation)
- Decoding requires: country code + location table number (LTN) + location code
- **Exception:** Netherlands uses VILD instead of standard Alert-C
- Some feeds may supply an incorrect LTN — ignore the `alertCLocationTableNumber` element and use the correct table directly

---

## Resources

**DATEX II Standard**
- [DATEX II Official Site](https://datex2.eu/)
- [DATEX Academy](https://datex2.eu/academy/)
- [DATEX II YouTube Channel](https://www.youtube.com/@datexii8761/)
- [Schema Generation Tool](https://webtool.datex2.eu/wizard/)

**Norway (NPRA)**
- [NPRA Open Data](https://www.vegvesen.no/en/fag/technology/open-data/)
- [Dataportalen (dataset index)](https://dataut.vegvesen.no/en/)
- [Transportportal](https://transportportal.no/)

**Sweden (Trafikverket)**
- [Trafikverket Open Data portal](https://data.trafikverket.se/)
- [DATEX II information page](https://bransch.trafikverket.se/en/startpage/operations/Operations-road/Traffic-information/Real-time-traffic-information/)
- [Trafiklab (API key for open API)](https://www.trafiklab.se/)

**Finland (Digitraffic / Fintraffic)**
- [Digitraffic Road Traffic](https://www.digitraffic.fi/en/road-traffic/)
- [Fintraffic Open Data](https://www.fintraffic.fi/en/fintraffic/open-data)

**Netherlands (NDW)**
- [NDW Open Data](https://opendata.ndw.nu/)
- [NDW Documentation](https://docs.ndw.nu/en/)

**Multi-country reference**
- [GraphHopper Open Traffic Collection](https://github.com/graphhopper/open-traffic-collection) — community-maintained list of open traffic data sources across Europe
