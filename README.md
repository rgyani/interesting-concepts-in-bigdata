# Consistent Hashing

In a large scale distributed system, we have multiple nodes holding the data.  
And we almost always have nodes being added or removed to scale the demand or maintenance or to improve the performance. This is called horizontal scaling.

In such scenarios we want to distribute the data as evenly as possible.

A common approach to determine which node gets which data is **Simple Hashing**.
For each incoming data, we hash the key (using an algo like Murmur Hash or FNV Hash which give an integer result) and take a modulo against the number of servers.

This approach works well when the number of servers remains constant. However, when this number gets changed, eg. a server is not available or a new server is added to improve performance, the rebalancing required to move the keys is a huge task.

Here is when consistent hashing comes into picture.

The central idea is, **we use a hash function that randomly maps both the data keys and servers to a unit circle**, usually `2 pi  radians`. 
So, as the diagram below, the 5 servers are placed in a ring (360 degrees) based on the hash of their ID or IP Address
Each **incoming data key is also then hashed and assigned to the next server that appears on the circle in clockwise order**

![img](imgs/Consistent_Hashing_Sample_Illustration.png)

This provides an even distribution of data to servers. But, more importantly, if a server fails and is removed from the circle, only the data(keys) that were mapped to the failed server need to be reassigned to the next server in clockwise order. Likewise, if a new server is added, it is added to the unit circle, and only the keys(data) mapped to that server need to be reassigned.

Importantly, when a server is added or removed, the vast majority of the keys(data) maintain their prior server assignments, and the addition of nth server only causes 1/n fraction of the keys(data) to relocate.

Some use cases include DynamoDB, Cassandra, CDN etc.

# Geohash


A Geohash is a unique identifier of a specific region on the Earth. The basic idea is that the Earth is divided into regions of user-defined size and each region is assigned a unique id, which is called its Geohash. For a given location on earth, the Geohash algorithm converts its latitude and longitude into a string.

A geohash actually identifies a rectangular cell: at each level, each extra character identifies one of 32 sub-cells.   
![img](imgs/geohash.jpg)

The cell sizes of geohashes of different lengths are as follows; note that the cell width reduces moving away from the equator (to 0 at the poles):


| Geohash length |	Cell width x Cell height |
|---|---|
|1	| ≤ 5,000km	×	5,000km |
|2	| ≤ 1,250km	×	625km |
|3	| ≤ 156km	×	156km |
|4	| ≤ 39.1km	×	19.5km |
|5	| ≤ 4.89km	×	4.89km |
|6	| ≤ 1.22km	×	0.61km |
|7	| ≤ 153m	×	153m |
|8	| ≤ 38.2m	×	19.1m |
|9	| ≤ 4.77m	×	4.77m |
|10| 	≤ 1.19m	×	0.596m |
|11| 	≤ 149mm	×	149mm | 
|12| 	≤ 37.2mm × 18.6mm |

Nearby locations generally have similar prefixes, though not always: there are edge-cases straddling large-cell boundaries;   
in France, La Roche-Chalais (u000) is just 30km from Pomerol (ezzz). 

A reliable prefix search for proximate locations will also search prefixes of a cell’s 8 neighbours.   
(e.g. a database query for results within 30-odd kilometres of Pomerol would be  
```
SELECT * FROM MyTable WHERE LEFT(Geohash, 4) IN ('ezzz', 'gbpb, 'u000', 'spbp', 'spbn', 'ezzy', 'ezzw', 'ezzx', 'gbp8'). 
```

Advantages of Geohash
1. Very easy to find the parent cell, by simply truncating the geohash
2. Very quick calculation to find the adjoining 8 cells
