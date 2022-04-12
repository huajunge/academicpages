---
title: "Handbook of JUST-Traj"
collection: teaching
type: "Handbook"
permalink: /teaching/2021-JUST-Traj
venue: "SIGSPATAIL"
date: 2021-11-01
video: ""
---

This is a Handbook of JUST-Traj.


<center>JUST-Traj: A Distributed and Holistic Trajectory Data Management System</center>

<center>HANDBOOK</center>



[TOC]

## DEMONSTRATION (Video)

<video controls="controls" src="https://huajunge.github.io/academicpages/files/just-traj-new.mp4"></video>

<center>Video</center>

## Web Poral

### Login

1. We prepare a test account for the reviewers. UserName: JustTestUser, Password:JustTestUser!1
2. Enter the login page: http://portal-just.urban-computing.cn/login
3. Enter the User Name (JustTestUser) and Password (JustTestUser!1), then click the button Login, as shown in the following picture.

<img src="https://huajunge.github.io/academicpages/images/tec/just-traj/login.png" alt="login" style="zoom:50%;" />

​                                                                                       Figure 1: Login.

### Web Poral

As shown in Figure 2, the web portal of JUST-Traj has three panels: (1) table panel, which manages the created tables; (2) SQL panel, which provides an SQL editor; and (3) result panel, which visualizes the result by multiple display forms, i.e., table view, chart view (i.e., histogram and line chart), and map view. 

<img src="https://huajunge.github.io/academicpages/images/tec/just-traj/portal.png" alt="portal" style="zoom:75%;" />

​                                                                       Figure 2: Web portal.

### Create Table

JUST-Traj uses the following statement to create a trajectory table: CREATE TABLE \<*table name*\> (\<*field name* \> Trajectory), where *field name* is the field name of the trajectory in the database. For example, 

```sql
CREATE TABLE traj_table (traj Trajectory)
```

### Load Data

We can load data from multiple sources into JUST-Traj using the following statement: LOAD \<source type\>:\<file path\> TO JUST:\<table name\> CONFIG {\<the field mapping relationship\>}; where *source type* could be HDFS, HIVE, KAFAK, FLINK, and the *CONFIG* provides the field mapping between the source and the JUST-Traj table. 

For example, we load data from HDFS to JUST-Traj.

```sql
LOAD HDFS:'/just_tutorial/trajectory_data' to JUST:traj_table ( 
traj.oid 0, 
traj.time to_timestamp(3), 
traj.point st_makePoint(1,2) 
);         
```

where `/trajectories' is the path of trajectories, lines from 2 to 4 are the field mappings.

### Query

JUST-Traj provides spatial or spatio-temporal queries: e.g., spatial range query, spatio-temporal range query, ID temporal query, kNN query.

(1) Spatial query: SELECT * FROM traj_table WHERE <spatial relationship>(traj, st_makeBBox(lng1,lat1, lng2, lat2); where, *spatial relationship* could be $st\_within$, $st\_intersect$, $st\_overlap$. *st\_makeBBox* is a spatial range formed by two points ($lng1,lat1$) and ($lng2, lat2$). 

For example,


```sql
select
    traj_linestring(traj) geom
from
    traj_table
where
    st_within(traj_linestring(traj), st_makeBBox(113.0, 23.0, 113.5, 23.6));
```

(2) Spatio-temporal query


```sql
SELECT * FROM traj_table
WHERE st_within(traj_linestring(traj), st_makeBBox(113.0, 23.0, 113.5, 23.6))
  and traj_startTime(traj) >= '2014-03-13 07:04:51'
  and traj_endTime(traj) <= '2014-03-16 08:04:51';
```

(3) ID-temporal query


```sql
SELECT * FROM traj_table
WHERE 
  traj_oid(traj) = '1003'
  and traj_startTime(traj) >= '2013-07-03 14:33:27'
  and traj_endTime(traj) <= '2018-08-03 14:33:27'
```

(4) Similarity Query

```sql
select
  *
from
  traj_table
where
  st_distanceWithin(
    traj,
    'LINESTRING (111.64161523718602 22.632099487568006,111.64196120470221 22.635834170510613,
      111.7091216719027 22.589810068829873)',
    'frechet',
    20000.0
  )
```

(5) K-NN query


```sql
SELECT
  *
FROM
  traj_table
WHERE
  st_knn(
    traj,
    'POINT(115.71 39.57)',
    'common',
    10
  )
```

### Analytics

JUST-Traj supports many out-of-the-box analyses on trajectories, which facilitates the development of applications. The SQL statement of DAL is as follows: SELECT \<analysis operation\>(traj, {\<parameters\>}) FROM \<table name\>; where, *analysis operation* is the name of analysis on trajectories, *parameters* set the corresponding parameters. Five common analyses provided by JUST-Traj, i.e.,

**Processing**. Although we can pre-process before storing, the parameter of each algorithm could be adjusted when analyzing. Thus, JUST-Traj also supports the processing in the analytics stage;

(1) Noise Filtering

```sql
SELECT
  st_trajnoisefilter(
     traj,
    '{ "filterType": "COMPLEX_FILTER",
      "maxSpeedMeterPerSecond": 100.0,
      "segmenterParams": { "maxTimeIntervalInMinute": 60,
      "maxStayDistInMeter": 100,
      "minStayTimeInSecond": 100,
      "minTrajLengthInKM": 1,
      "segmenterType": "ST_DENSITY_SEGMENTER"}}'
  )
FROM
  traj_table
LIMIT 1000
```

(2) Segmentation

```sql
SELECT st_trajSegmentation(traj,
    '{ "maxTimeIntervalInMinute": 10,
      "maxStayDistInMeter": 100,
      "minStayTimeInSecond": 100,
      "minTrajLengthInKM": 1,
      "segmenterType": "HYBRID_SEGMENTER"
    }'
  )
FROM traj_table
LIMIT 1000
```

(3) Interpolation

```sql
SELECT st_freespaceInterpolation(
    traj, 30)
FROM traj_table
LIMIT 100
```

(4) MapMatching

create a trajectory table for MapMatching

```sql
CREATE TABLE mm_traj (traj Trajectory)
```

Load Data

```sql
LOAD hdfs: '/just_ci_data/mm_traj.csv' to just: mm_traj(
 traj traj_recover(traj)
)WITH (
"just.separator"="|",
"just.header"="true"
)
```

create a road network table

```sql
create table if not exists longgang_expand_rn (
  road RoadSegment,
  weight Double
);
```

Load Data

```sql
LOAD hdfs: '/just_ci_data/longgang_expand_rn.csv' to just: longgang_expand_rn (
          road.oid oid,
          road.geom st_lineFromText(geom),
          road.direction direction,
          road.level level,
          road.speed_limit speed_limit,
          weight weight
) WITH (
"just.separator"="|",
"just.header"="true"
)
```

MapMatching

ST_trajMapMatchToProjection 

```sql
select 
 ST_trajMapMatchToProjection(
 t1.traj,t2.t
 )
from
 mm_traj t1,
 (select ST_makeRoadNetwork(collect_list(road))as t FROM longgang_expand_rn) as t2
```

ST_trajMapMatchToRoute

```sql
SELECT
  st_trajMapMatchToRoute(
    t1.traj,
    t2.t
  )
FROM
  mm_traj t1,
 (SELECT st_makeRoadNetwork(collect_list(road))as t FROM longgang_expand_rn) AS t2
```

**Aggregation**. JUST-Traj provides many aggregation operations, e.g., $max()$, $min()$, $count()$; For example,

```sql
SELECT count(*) FROM traj_table;
```

**Stay Point Detection.** Moving objects tend to stay due to certain events, such as vehicles staying for refueling, couriers staying for delivery. By analyzing the place that a mobile object stays, we can infer to some places of interesting, e.g., the delivery address;

```sql
SELECT st_trajStayPoint(traj,
  '{ "maxStayDistInMeter": 10,
     "minStayTimeInSecond": 60}')
FROM
  traj_table
LIMIT 1000
```

### Holistic Solutions

**Spatio-temporal Query and Stay Point Detection** 
In this scenario, we detect stay points from the spatio-temporal query result. We define the location where the lorry stays over a given time threshold (*minStayTimeInSecond*) as a stay point, where the location is a spatial region whose maximum distance not greater than a distance threshold (*maxStayDisInMeter*). The underlying locations of stay points could be the delivery addresses. The SQL is as follows:

```sql
SELECT st_trajStayPoint(traj,
  '{ "maxStayDistInMeter": 10,
     "minStayTimeInSecond": 60}')
FROM
  traj_table
WHERE
  st_within(traj_linestring(traj), 
      st_makeBBox(113.0, 23.0, 113.5, 23.6))
  and traj_startTime(traj) >= '2014-03-13 07:04:51'
  and traj_endTime(traj) <= '2014-03-16 08:04:51'
```

Lines from 7 to 10 take a spatio-temporal range to query trajectories from the database. Lines from 1 to 3 execute the *Stay Point Detection* operation on the extracted trajectories, where lines from 2 to 3 are parameters of *Stay Point Detection*. 

**ID Temporal Query and Noise Filtering**

In this scenario, we define the point whose speed exceeds the maximum limited speed (*maxSpeedMeterPerSecond*) as the noise point, and we filter that point. The SQL is as follows:

LOADING trajectories of Guiyang:

```sql
LOAD HDFS :'/just_tutorial/guiyang_traj' to JUST :traj_table (
  traj.oid 1,
  traj.time to_timestamp(0),
  traj.point st_makePoint(3, 2)
)
```

The raw trajectory

```sql
SELECT
  traj_linestring(traj)
FROM
  traj_table
WHERE
  traj_oid(traj) = '1192408782';
```

<img src="https://huajunge.github.io/academicpages/images/tec/just-traj/image-20210601200321686.png" alt="image-20210601200321686" />

<center>Figure 3: The raw trajectory of '1192408782'.</center>

Noise Filtering

```sql
SELECT
  traj_linestring(aa.item)
from
  (
    SELECT
      st_trajnoisefilter(
        traj,
        '{ "filterType": "COMPLEX_FILTER",
      "maxSpeedMeterPerSecond": 20.0,
      "segmenterParams": { "maxTimeIntervalInMinute": 10,
      "maxStayDistInMeter": 100,
      "minStayTimeInSecond": 100,
      "minTrajLengthInKM": 1,
      "segmenterType": "ST_DENSITY_SEGMENTER"}}'
      )
    FROM
      traj_table
    WHERE
      traj_oid(traj) = '1192408782'
  and traj_startTime(traj) >= '2018-06-03 14:33:27'
  and traj_endTime(traj) <= '2018-08-03 14:33:27'
  ) as aa
```

Lines from 7 to 9 take a *ID Temporal* query to extract trajectories from the database. Lines from 1 to 3 execute the *Noise Filtering* operation on the extracted trajectories.

![image-20210601200416279](images/image-20210601200416279.png)

<center>Figure 3: Cleaned Trajectories of '1192408782'.</center>



### Download Back Results

```sql
DOWNLOAD
SELECT
  st_trajStayPoint(
    traj,
    '{ "maxStayDistInMeter": 10,      "minStayTimeInSecond": 60}'
  )
FROM
  traj_table
WHERE
  st_within(
    traj_linestring(traj),
    st_makeBBox(113.0, 23.0, 113.5, 23.6)
  )
  and traj_startTime(traj) >= '2014-03-13 07:04:51'
  and traj_endTime(traj) <= '2014-03-16 08:04:51'
```

Results

![image-20210727101710204](images/image-20210727101710204.png)



## DB-Driver

### Download DB-Driver

The DB Driver is a Java-based API for accessing JUST-Traj DB. It complies with the JDBC specification, implements most of the interfaces, and extends some space-time attributes.  Users can download the just-drive-java.jar from http://nexus.cbpmgt.com/nexus/#nexus-search;quick~just-driver-java.

![image-20210726180740968](images/image-20210726180740968.png)

### Quick Start

Because the API follow the JDBC specification, so we can according to the JDBC API to access DB. The official document of JDBC: https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html.

**Parameters**

- The first is the JDBC URL. JUST begins with JDBC: JUST: followed by a parameter in the form key=value;

- Note: Urls must be configured = XXX;

- There are three ways to pass a parameter. For example, access to JUST requires user key authentication:

  - Specify the username secret key in the URL: jdbc:just:urls=https://portal-just.urban-computing.cn/server2;user=xxx;privateKey=xxx;

  - Specify by code;

    ```java
    Use native Properties to set parameters: JustProperties essentially just inherits Properties and extends a few methods. When using Properties, users need to pay attention to whether the key of these parameters changes, so it is not recommended
    ```

  - Use native Properties to set parameters: JustProperties essentially just inherits Properties and extends a few methods. When using Properties, users need to pay attention to whether the key of these parameters changes, so it is not recommended.

### Demo

Here's a demo to use DB-Driver

```java
final JustProperties p = new JustProperties("xxx", "sec");
p.setDB("my_db");
p.setPageSize(1000);
try (final Connection connection = DriverManager.getConnection("jdbc:just:urls=http://xxxx", p)) {
    final Statement statement = connection.createStatement();
    final ResultSet rs = statement.executeQuery("select attr1,attr2,attr3 as a3 from order_table");
    final ResultSetMetaData md = rs.getMetaData();
    final int columnCount = md.getColumnCount();
    assertEquals(3, columnCount);
    while (rs.next()) {
        assertNotNull(rs.getString(1));
        assertNotNull(rs.getString(2));
        assertNotNull(rs.getString(3));
        assertNotNull(rs.getString("attr1"));
        assertNotNull(rs.getString("attr2"));
        assertNotNull(rs.getString("a3"));
    }
}
```

### For trajectory

**Create table**

```java
String sql  = "CREATE TABLE traj_table (traj Trajectory)";
Statement stmt = connection.createStatement();
stmt.executeUpdate(sql);
```

**Load Data**

```sql
String sql  = "LOAD HDFS:'/just_tutorial/trajectory_data' to JUST:traj_table (trajectory.oid 0,    trajectory.time to_timestamp(3),    trajectory.point st_makePoint(1,2)); ";
Statement stmt = connection.createStatement();
stmt.executeUpdate(sql);
```

**Query**

```java
String sql = "sql";
final Statement st = connection.createStatement();
final JustResultSet rs = (JustResultSet) st.executeQuery(sql);
```

where "sql" is the query statement provided by JUST-Traj. e.g., for spatial query

```java
String sql = "select traj_linestring(traj) geom from traj_table where    st_within(traj_linestring(traj), st_makeBBox(113.0, 23.0, 113.5, 23.6))";
final Statement st = connection.createStatement();
final JustResultSet rs = (JustResultSet) st.executeQuery(sql);
while (rs.hasNext()) {
    System.out.println(rs.next().getString(1));
}
```

other queries can refer to the statements described in Web portal.