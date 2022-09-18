# IpGeoFencing
A simple and easily extensible IP geo-fencing engine

## Overview
Geo-fencing is the use of virtual geographic perimeters to determine the behavior of an application.
Provided a specific coordinate (latitude and longitude), a geo-fencing engine can determine to which virtal geographic area or areas the location belongs, and consequentially take action.

The IP geo-fencing engine in this project run based on specific IP addresses.
The engine would first translate the IP address into a geographic location, using the free CSV database provided by IP2Location
[CSV database](https://lite.ip2location.com/database/db11-ip-country-region-city-latitude-longitude-zipcode-timezone "[https://geojson.org/](https://lite.ip2location.com/database/db11-ip-country-region-city-latitude-longitude-zipcode-timezone)") (included in the project).

The engine then determines which of the configured geographic areas contain the location, and for each match fires the correspondent configured action.
Geographic areas can be configured in the engine by importing a file in [GeoJSON format](https://geojson.org/ "https://geojson.org/").
Sample GeoJSON files are included in this project for testing purposes, and also freely available for the public on the [OpenDataSoft website](https://public.opendatasoft.com/explore/ "https://public.opendatasoft.com/explore/").

## Use case scenarios
This engine is particularly useful in cases when the ip address is available, but not the coordinate.<br />
These are the typical scenarios in which web applications can use the engine with the ip address of the http requests:

- Blocking requests from black-listed geographic areas
- Enabling/disabling features for specific regions
- Localizing content
- Finding the availability of products or service nearby the ip location

## Configuring the geo-fencing engine
```csharp
var engine = new IPGeoFencingEngineBuilder()
	.AddIP2LocationFromCSVFile(@"\\geo-fencing\data\IP2LOCATION-LITE-DB11.CSV")
	.AddGeographicAreasFromGeoJSONFile(@"\\geo-fencing\data\demo.geojson")
	.AddRule("New York", 
		predicate: (areas, ip, location) => { return areas.Any(A => A.Name == "New York"); },	
		action: (areas, ip, location) => { Console.WriteLine($"The IP Address: {ip} is in New York State!"); })
	.AddRule("Montana",
		predicate: (areas, ip, location) => { return areas.Any(A => A.Name == "Montana"); },
		action: (areas, ip, location) => { Console.WriteLine($"The IP Address: {ip} is in Montana!"); })
	.AddRule("Billings",
		predicate: (areas, ip, location) => { return areas.Any(A => A.Name == "Billings"); },
		action: (areas, ip, location) => { Console.WriteLine($"The IP Address: {ip} is in Billings, MT"); })
	.AddRule("Montana but not Billings",
		predicate: (areas, ip, location) => { return areas.Any(A => A.Name == "Montana") && !areas.Any(A => A.Name == "Billings"); },
		action: (areas, ip, location) => { Console.WriteLine($"The IP Address: {ip} is in Montana but not in Billings!"); })
	.AddRule("New York or Montana",
		predicate: (areas, ip, location) => { return areas.Any(A => A.Name == "Montana") || areas.Any(A => A.Name == "New York"); },
		action: (areas, ip, location) => { Console.WriteLine($"The IP Address: {ip} is in New York State or Montana!"); })
	.AddDefaultAction((ip) => Console.WriteLine($"The IP Address: {ip} is outside all the areas provided"))
	.Build();
```
## Running the engine
```csharp

engine.Run("98.127.147.57"); //Billings, MT IP Address
//The IP Address: 98.127.147.57 is in Montana!
//The IP Address: 98.127.147.57 is in Billings, MT
//The IP Address: 98.127.147.57 is in New York State or Montana!

engine.Run("172.254.112.210"); //New York, NY IP Address
//The IP Address: 172.254.112.210 is in New York State!
//The IP Address: 172.254.112.210 is in New York State or Montana!


engine.Run("157.240.3.35"); //Seattle, WA IP Address
//The IP Address: 157.240.3.35 is outside all the areas provided
```
