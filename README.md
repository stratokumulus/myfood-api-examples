# Myfood API examples

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
