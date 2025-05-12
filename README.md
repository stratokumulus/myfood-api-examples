# Myfood API examples

# First, we must authenticate

To get started with the myfood API, you will need your greenhouse ID, your username, and your password (both received when your ordered your greenhouse). 

With that, run on your server (I'm using ZSH, if you're using anything else, Google is your friend :) )

```bash
curl -X POST https://hub.myfood.eu/api/identity/token -d '{"userName":"<put the name between double quotes>", "password": "<put the password between double quotes>"}' -H 'Content-Type: application/json'
```

Copy the whole token value : 
```
{"data":{"token":"a mega long string that must be completely copied",'"refreshToken":"..." ...}
```

Let's call this TOKEN. 

Grab the info from your greenhouse using the following curl command (replace TOKEN and GREENHOUSE_ID by the proper values) :
```bash
curl "https://hub.myfood.eu/api/v1/ProductionUnit/GetProductionUnitDetailForUser?id=GREENHOUSE_ID" -X GET -H 'Authorization: Bearer 'TOKEN' -H 'Accept: */*'
```

This will generate an output like :
```json
{
  "data":
  {
    "pioneerCitizenName":"your myfood name",
    "pioneerCitizenNumber":your_ID_pioneer_number,
    "productionUnitVersion":null,
    "productionUnitType":"blah",
    "picturePath":"blah",
    "productionUnitOptions":"",
    "onlineSinceWeeks":blah,
    "averageMonthlyProduction":blah,
    "averageMonthlySparedCO2":blah,
    "currentPhValue":blah,
    "currentPhCaptureTime":"blah",
    "averageHourPhValue":blah,
    "averageDayPhValue":blah,
    "lastDayPhCaptureTime":"blah",
    "currentWaterTempValue":blah,
    "currentWaterTempCaptureTime":"blah",
    "averageHourWaterTempValue":blah,
    "averageDayWaterTempValue":blah,
    "lastDayWaterTempCaptureTime":"blah",
    "currentAirTempValue":blah,
    "currentAirTempCaptureTime":"blah",
    "averageHourAirTempValue":blah,
    "averageDayAirTempValue"blah,
    "lastDayAirTempCaptureTime":"blah",
    "currentHumidityValue":blah,
    "currentHumidityCaptureTime":"blah",
    "averageHourHumidityValue":blah,
    "averageDayHumidityValue":blah,
    "lastDayHumidityCaptureTime":"blah",
    "lastSignalStrenghtReceived":"Average"
  },
  "failed":false,
  "messages":[],
  "succeeded":true,
"type":null}
```

If you see this, it means that you can access the REST API of the MyFood hub with your username and password.

## Let's use this in Home Assistant

Now, we'll use these values to automate the authentication and collection of data. 

NOTE: when booting Home Assistant, allow 15 minutes (the 900 seconds defined in the configuration) to values to be read, a.k.a. it's going to show unavailable sensor for 15 minutes after the boot (or any restart of HA)

In the file `secrets.yaml`, configure the user name and password you used in the first CURL command: 

```
myfood_payload: '{"userName": "prosper", "password": "YouplaBoum"}'
```
Configure two rest entries in `configuration.yaml` : 
- one that will create a sensor that contains the token
```
  - resource: "https://hub.myfood.eu/api/identity/token"
    method: POST
    payload: !secret myfood_payload
    scan_interval: 43200
    headers:
      Content-Type: application/json
    sensor:
      - name: "gtoken"
        value_template: "{{ value_json.data.refreshToken }}"
        json_attributes_path: $.data
        json_attributes:
          - token
          - refreshTokenExpiryTime
```
- one that will create a sensor that contains the greenhouse details
```
  - resource: "https://hub.myfood.eu/api/v1/ProductionUnit/GetProductionUnitDetailForUser?id=942"
    method: GET
    headers:
      Authorization: "Bearer {{ state_attr('sensor.gtoken','token')}}"
      Accept: "*/*"
    scan_interval: 900
    sensor:
      - name: greenhouse
        value_template: "{{ value_json.data.pioneerCitizenNumber }}"
        json_attributes_path: $.data
        json_attributes:
          - currentPhValue
          - currentWaterTempValue
          - currentAirTempValue
          - currentHumidityValue
```

For this last one, select the attributes you want to use in HA. I decided to only use the real time values and not historical ones. 

In the file `templates.yaml`, create a new sensor for each of the attributes : 

```
- sensor:
    - name: greenhouse_current_air_temp
      state: "{{ state_attr ('sensor.greenhouse','currentAirTempValue') }}"
```

You can now use `greenhouse_current_air_temp` as an entity : 

![](./tile%20card%20config.png)