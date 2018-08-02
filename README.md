# Merging-and-aggregating-json-datasets-in-pig-to-calculate-number-of-bikes-available-in-Toronto
Merging two datasets that are stored in json files. Calculating % of bikes available on bike stations in Toronto. Pig code for a reference

Datasets available [here](https://www.toronto.ca/city-government/data-research-maps/open-data/open-data-catalogue/#84045f23-7465-0892-8889-7b6f91049b29)


**Load  json files with JsonLoader**

```
station_information = LOAD '/user/hirwuser864/bikes_input/bikes/station_information.json' USING JsonLoader('station_id:int, name:chararray, lat:float, lon:float, address:chararray, capacity:int, rental_methods:{(items:chararray)}');

station_status = LOAD '/user/hirwuser864/bikes_input/bikes/station_status.json' USING JsonLoader('station_id:int, num_bikes_available:int, num_bikes_disabled:int, num_docks_available:int, num_docks_disabled:int, is_installed:int, is_renting:int, is_returning:int, last_reported:long');
```

**Merging datasets by station_id**

```
join_inner = JOIN station_information BY (station_id) , station_status BY (station_id);


join_project  = FOREACH join_inner GENERATE station_information::station_id, station_information::address, station_information::capacity, station_status::num_bikes_available, station_status::num_docks_available, station_status::num_docks_disabled, station_information::lat, station_information::lon;
```

**Calculating number of bikes available for every station**

```
join_project_f  = FOREACH join_project GENERATE 
   station_information::station_id as station_id,
   station_information::address as address,
   station_information::capacity as capacity,
   station_status::num_bikes_available as num_bikes_available,
   station_status::num_docks_available as num_docks_available,
   station_status::num_docks_disabled as num_docks_disabled,
   1-((float)station_information::capacity-(float)station_status::num_bikes_available)/(float)station_information::capacity  as percent_bikes,
   station_information::lat as lat,
   station_information::lon as lon;
```

**Saving result as json file with JsonStorage**

```
STORE join_project_f INTO '/user/hirwuser864/bikes_output/output.json' USING JsonStorage();
```

**Subset of output**
>{"station_id":7000,"address":"Fort York  Blvd / Capreol Crt","capacity":31,"num_bikes_available":31,"num_docks_available":0,"num_docks_disabled":0,"percent_bikes":1.0,"lat":43.63983,"lon":-79.39595}
>{"station_id":7078,"address":"College St / Major St","capacity":11,"num_bikes_available":8,"num_docks_available":2,"num_docks_disabled":0,"percent_bikes":0.72727275,"lat":43.6576,"lon":-79.4032}
