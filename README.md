# ddolsoon maps platform 

외부 Vendor API에 의존하지 않고, 독립적인 지도 관련 서비스 기능을 제공하기 위해서 만든 지도 API 플랫폼 입니다. 

**[host] : ddolsoon.com**

최대 TPS 제한 : 5 TPS (5 TPS 초과시, QoS 모듈에 의해서 429 에러 반환)

**제공되는 API 목록**

1. Reverse Geocoding (리버스 지오코딩 API)
2. Coordinate Transform API (좌표계 변환 API)
3. Walking Route Search API (도보 길찾기 API)
4. Car Route Search API (자동차 길찾기 API)
5. Bicycle Route Search API (자전거 길찾기 API)
6. Nearest Subway Search API (가까운 지하철 조회 API)



## 1. Reverse Geocoding (리버스 지오코딩 API)

지리적좌표(위경도 좌표 epsg:4326)를 우리나라 도로명/지번 주소로 변환해주는 API

### Request

#### Request URL

```
GET http://[host]//map-geocode/v1/reverse-geocoding.api
```

#### Query Parameters

| Name | Type   | Mandatory | Description |
| :--- | :----- | :-------- | :---------- |
| lat  | Double | N         | 위도        |
| lon  | Double | N         | 경도        |

### Error Code

| code | message                 | Description                           |
| :--- | :---------------------- | :------------------------------------ |
| 0    | success                 | 성공                                  |
| 2004 | Not Exists Adm District | 검색결과의 행정구역이 존재하지 않을때 |

## Response

### Success

##### Status 200

```
{
    "code": 0,
    "message": "success",
    "result": {
      "sido" : "서울특별시",
      "sigungu" : "용인시",
      "eupmyundong" : "똘순동",
      "address" : "서울특별시 용인시 똘순동"
    }
}
```

| Name        | Type   | Description |
| :---------- | :----- | :---------- |
| sido        | String | 시/도       |
| sigungu     | String | 시/군/구    |
| eupmyundong | String | 읍/면/동    |
| address     | String | 전체 주소   |



## 2. Coordinate Transform API (좌표계 변환 API)

다양한 좌표계 변환을 제공하는 API

**epsg:5179 : GRS80 타원체의 UTM-K 직각좌표계로, 주로 네이버와 도로명 주소 DB에 사용되는 좌표계**

**epsg:4326 : WGS84 타원체의 경위도 좌표게 ( 흔히 쓰는 GPS 좌표계 )**

**보정된 중부원점(Bessel) : KLIS에서 중부지역에서 사용중 ( 공공데이터 사용시, 일부 데이터들이 이 좌표계를 사용함 )**

**현재 지원되는 좌표계 변환 목록**

| **입력 좌표계**          | **변환 좌표계** |
| :----------------------- | :-------------- |
| epsg5179                 | epsg4326        |
| epsg4326                 | epsg5179        |
| 보정된 중부원점 (Bessel) | epsg4326        |

### Request

#### Request URL

```
GET http://[host]//map-coordinate/v1/transformCoordinate.api
```

#### Query Parameters

| Name          | Type   | Mandatory | Description                                                |
| :------------ | :----- | :-------- | :--------------------------------------------------------- |
| inputX        | Double | Y         | 위도 or UTM-K의 x좌표                                      |
| inputY        | Double | Y         | 경도 or UTM-K의 y좌표                                      |
| inCoordinate  | String | Y         | 입력 좌표계<br />epsg5179<br />epsg4326<br />besselcentorg |
| outCoordinate | String | Y         | 변환 좌표계<br />epsg5179<br />epsg4326                    |

### Error Code

| code | message                            | Description                      |
| :--- | :--------------------------------- | :------------------------------- |
| 0    | success                            | 성공                             |
| 2006 | Not Supported Coordinate Transform | 지원되지 않는 좌표계 변환인 경우 |

## Response

### Success

##### Status 200

```
{
  "code": 0,
  "message": "success",
  "result": {
    "outputX": 980093.4093619981,
    "outputY": 1830311.648982822
  }
}

{
  "code": 0,
  "message": "success",
  "result": {
    "outputX": 36.47032034568296,
    "outputY": 127.27808796395965
  }
}
```

| Name    | Type   | Description                    |
| :------ | :----- | :----------------------------- |
| outputX | Double | 변환된 (위도 or UTM-K의 x좌표) |
| outputY | Double | 변환된 (경도 or UTM-K의 y좌표) |

## 3. Walking Route Search API (도보 길찾기 API)

출발지(from) 에서 목적지(to)까지의 도보(walking)로 갈 수 있는 최단 경로 및 거리/걸린시간을 가져오는 API

### Request

#### Request URL

```
GET http://[host]/map-routing/v1/walking/distance.api
```

#### Query Parameters

| Name    | Type   | Mandatory | Description |
| :------ | :----- | :-------- | :---------- |
| fromLat | Double | Y         | 출발지 위도 |
| fromLon | Double | Y         | 출발지 경도 |
| toLat   | Double | Y         | 목적지 위도 |
| toLon   | Double | Y         | 목적지 경도 |

### Error Code

| code  | message           | Description         |
| :---- | :---------------- | :------------------ |
| 0     | success           | 성공                |
| 10000 | Osrm Engine Error | OSRM 엔진 관련 에러 |

## Response

### Success

##### Status 200

```
{
    "code": 0,
    "message": "success",
    "result": {
        "summary": "경기도",
        "distance": 1444,
        "duration": 1039.5,
        "routes": [
            {
            "lat": 37.39551,
            "lon": 127.128272
            },
            {
            "lat": 37.394894,
            "lon": 127.128283
            },
            ....
        [
    }
}
```

| Name       | Type            | Description                                           |
| :--------- | :-------------- | :---------------------------------------------------- |
| summary    | String          | 최단 경로 요약본                                      |
| distance   | Integer         | 도보로 출발지에서 목적지까지의 걸리는 거리 (단위 : m) |
| duration   | Double          | 도보로 출발지에서 목적지까지의 걸리는 시간 (단위 : s) |
| routes     | Array\<Object\> | 출발 위치에서 목적지 까지의 도보 최단 경로            |
| routes.lat | Double          | 위도                                                  |
| routes.lon | Double          | 경도                                                  |



## 4. Car Route Search API (자동차 길찾기 API)

출발지(from) 에서 목적지(to)까지의 자동차(car)로 갈 수 있는 최단 경로 및 거리/걸린시간을 가져오는 API

### Request

#### Request URL

```
GET http://[host]/map-routing/v1/car/distance.api
```

#### Query Parameters

| Name    | Type   | Mandatory | Description |
| :------ | :----- | :-------- | :---------- |
| fromLat | Double | Y         | 출발지 위도 |
| fromLon | Double | Y         | 출발지 경도 |
| toLat   | Double | Y         | 목적지 위도 |
| toLon   | Double | Y         | 목적지 경도 |

### Error Code

| code  | message           | Description         |
| :---- | :---------------- | :------------------ |
| 0     | success           | 성공                |
| 10000 | Osrm Engine Error | OSRM 엔진 관련 에러 |

## Response

### Success

##### Status 200

```
{
    "code": 0,
    "message": "success",
    "result": {
        "summary": "경기도",
        "distance": 1444,
        "duration": 1039.5,
        "routes": [
            {
            "lat": 37.39551,
            "lon": 127.128272
            },
            {
            "lat": 37.394894,
            "lon": 127.128283
            },
            ....
        [
    }
}
```

| Name       | Type            | Description                                             |
| :--------- | :-------------- | :------------------------------------------------------ |
| summary    | String          | 최단 경로 요약본                                        |
| distance   | Integer         | 자동차로 출발지에서 목적지까지의 걸리는 거리 (단위 : m) |
| duration   | Double          | 자동차로 출발지에서 목적지까지의 걸리는 시간 (단위 : s) |
| routes     | Array\<Object\> | 출발 위치에서 목적지 까지의 자동차 최단 경로            |
| routes.lat | Double          | 위도                                                    |
| routes.lon | Double          | 경도                                                    |

## 5. Bicycle Route Search API (자전거 길찾기 API)

출발지(from) 에서 목적지(to)까지의 자전거(bicycle)로 갈 수 있는 최단 경로 및 거리/걸린시간을 가져오는 API

### Request

#### Request URL

```
GET http://[host]/map-routing/v1/bicycle/distance.api
```

#### Query Parameters

| Name    | Type   | Mandatory | Description |
| :------ | :----- | :-------- | :---------- |
| fromLat | Double | Y         | 출발지 위도 |
| fromLon | Double | Y         | 출발지 경도 |
| toLat   | Double | Y         | 목적지 위도 |
| toLon   | Double | Y         | 목적지 경도 |

### Error Code

| code  | message           | Description         |
| :---- | :---------------- | :------------------ |
| 0     | success           | 성공                |
| 10000 | Osrm Engine Error | OSRM 엔진 관련 에러 |

## Response

### Success

##### Status 200

```
{
    "code": 0,
    "message": "success",
    "result": {
        "summary": "경기도",
        "distance": 1444,
        "duration": 1039.5,
        "routes": [
            {
            "lat": 37.39551,
            "lon": 127.128272
            },
            {
            "lat": 37.394894,
            "lon": 127.128283
            },
            ....
        [
    }
}
```

| Name       | Type            | Description                                             |
| :--------- | :-------------- | :------------------------------------------------------ |
| summary    | String          | 최단 경로 요약본                                        |
| distance   | Integer         | 자전거로 출발지에서 목적지까지의 걸리는 거리 (단위 : m) |
| duration   | Double          | 자전거로 출발지에서 목적지까지의 걸리는 시간 (단위 : s) |
| routes     | Array\<Object\> | 출발 위치에서 목적지 까지의 자전거 최단 경로            |
| routes.lat | Double          | 위도                                                    |
| routes.lon | Double          | 경도                                                    |



## 6. Nearest Subway Search API (가까운 지하철 조회 API)

위경도(lat/lon) 위치의 반경(distance)기준으로 가장 가까운 지하철 정보를 가져온다.

### Request

#### Request URL

```
GET http://[host]/map-routing/v1/subway/getNearestSubway.api
```

#### Query Parameters

| Name      | Type    | Mandatory | Description                     |
| --------- | ------- | --------- | ------------------------------- |
| latitude  | Double  | Y         | 위도                            |
| longitude | Double  | Y         | 경도                            |
| distance  | Integer | N         | 반경(단위 : (m)) default : 1000 |

### Error Code

| code  | message            | Description                                 |
| ----- | ------------------ | ------------------------------------------- |
| 0     | success            | 성공                                        |
| 2001  | Exists Near Subway | 반경에서 근방의 지하철이 존재하지 않는 경우 |
| 10000 | Osrm Engine Error  | OSRM 엔진 관련 에러                         |



## Response

### Success

##### Status 200

```
{
    "code": 0,
    "message": "success",
    "result": {
        "city": "경기도",
        "distance": 1444,
        "dong": "서현동",
        "duration": 1039.5,
        "latitude": 37.38448659,
        "longitude": 127.1227456,
        "subCity": "성남시 분당구",
        "subway": "서현역",
        "subwayId": "50004",
        "routes": [
            {
            "lat": 37.39551,
            "lon": 127.128272
            },
            {
            "lat": 37.394894,
            "lon": 127.128283
            },
            ....
        [
    }
}
```

| Name       | Type            | Description                                 |
| ---------- | --------------- | ------------------------------------------- |
| subwayId   | String          | 지하철 ID                                   |
| city       | String          | 시                                          |
| subCity    | String          | 구                                          |
| dong       | String          | 동                                          |
| subway     | String          | 지하철 명                                   |
| latitude   | Double          | 지하철 위도 (epsg:4326)                     |
| longitude  | Double          | 지하철 경도 (epsg:4326)                     |
| distance   | Integer         | 도보로 지하철 까지의 거리 (단위 : m)        |
| duration   | Double          | 도보로 지하철 까지의 걸리는 시간 (단위 : s) |
| routes     | Array\<Object\> | 현재 위치에서 지하철역 까지의 경로          |
| routes.lat | Double          | 위도                                        |
| routes.lon | Double          | 경도                                        |