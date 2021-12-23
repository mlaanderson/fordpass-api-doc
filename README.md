# FordPass API
Documentation of the FordPass API

# About
This api has been gleaned from various implementations, most of which trace their ancestry to [d4v3y0rk's ffpass](https://github.com/d4v3y0rk/ffpass)

# Conventions
When variables need to be included in a URL, the variable positions will be indicated with an all caps variable name
```javascript
"https://example.com/request/path?access_token=ACCESS_TOKEN"
```
When variables need to be indicated in a JSON structure, a representative variable name will be used. e.g.
```json
{
    "auth-token": token.access_token
}
```

# API

## Authentication
FordPass uses a token based authentication very similar to an OAUTHv2 token. 

### Initial Token
To request the initial token, POST a login request to https://fcis.ice.ibmcloud.com/v1.0/endpoint/default/token. The headers need to be:
```json
{
    "User-Agent": "fordpass-na/353 CFNetwork/1121.2.2 Darwin/19.3.0",
    "Content-Type": "application/x-www-form-urlencoded"
}
```
The JSON body of the request will be:
```json
{
    "client_id": "9fb503e0-715b-47e8-adfd-ad4b7770f73b", // This is the client ID that represents the FordPass application
    "grant_type": "password", // This is always "password", it specifies the type of login
    "username": "...",        // This is the vehicle owner's account name
    "password": "..."         // This is the vehicle owner's password
}
```
The JSON response will be like:
```json
{
    "access_token": "...",  // String value to be passed with every request
    "refresh_token": "...", // String value to request a new access token if it has expired
    "grant_id": "...",      // Unused
    "token_type": "jwt",    // Unused
    "expires_in": 300       // Time in seconds until the access_token expires
}
```

### Refresh Token
To refresh an expired token, POST a refresh_token request to https://fcis.ice.ibmcloud.com/v1.0/endpoint/default/token. The headers need to be:
```json
{
    "User-Agent": "fordpass-na/353 CFNetwork/1121.2.2 Darwin/19.3.0",
    "Content-Type": "application/x-www-form-urlencoded"
}
```
The JSON body of the request will be:
```json
{
    "client_id": "9fb503e0-715b-47e8-adfd-ad4b7770f73b", // This is the client ID that represents the FordPass application
    "grant_type": "refresh_token", // This is always "refresh_token", it specifies the type of login
    "refresh_token": "..."         // This is the refresh token returned from the previous token request
}
```
The JSON response will be like:
```json
{
    "access_token": "...",  // String value to be passed with every request
    "refresh_token": "...", // String value to request a new access token if it has expired
    "grant_id": "...",      // Unused
    "token_type": "jwt",    // Unused
    "expires_in": 300       // Time in seconds until the access_token expires
}
```

## Making a Vehicle Request
Vehicle requests are HTTP requests to various endpoint URLs. FordPass uses GET, POST, PUT, and DELETE HTTP methods.

Most vehicle requests _must_ contain the VIN in the URL, and that VIN must be associated with the user's FordPass account.

When making a request, the headers must be:
```json
{
    "Content-Type": "application/json",
    "User-Agent": "FordPass/5 CFNetwork/1327.0.4 Darwin/21.2.0",
    "Application-Id": "71A3AD0A-CF46-4CCF-B473-FC7FE5BC4592",
    "auth-token": authentication_response.access_token
}
```

For POST methods, a JSON body will usually be included. The body must be stringified JSON.

Vehicle commands that can be reversed, like LOCK/UNLOCK will often use the same end point URL with PUT and DELETE methods. For instance the LOCK command will be a PUT to https://usapi.cv.ford.com/api/vehicles/VIN/doors/lock, while the UNLOCK command will be a DELETE to the same endpoint.

## Commands

### Lock
```
PUT https://usapi.cv.ford.com/api/vehicles/VIN/doors/lock
```

### Unlock
```
DELETE https://usapi.cv.ford.com/api/vehicles/VIN/doors/lock
```

### Start
```
PUT https://usapi.cv.ford.com/api/vehicles/VIN/engine/start
```

### Stop
```
DELETE https://usapi.cv.ford.com/api/vehicles/VIN/engine/start
```

### Zone Lighting On (All Lights)
```
PUT https://usapi.cv.ford.com/api/vehicles/VIN/zonelightingactivation
PUT https://usapi.cv.ford.com/api/vehicles/VIN/0/zonelighting
```

### Zone Lighting Off (All Lights)
```
DELETE https://usapi.cv.ford.com/api/vehicles/VIN/zonelightingactivation
DELETE https://usapi.cv.ford.com/api/vehicles/VIN/0/zonelighting
```

### SecuriAlert Enable
```
PUT https://api.mps.ford.com/api/guardmode/v1/VIN/session
```

### SecuriAlert Disable
```
DELETE https://api.mps.ford.com/api/guardmode/v1/VIN/session
```

### Trailer Light Check On
```
PUT https://usapi.cv.ford.com/api/vehicles/VIN/trailerlightcheckactivation
```

### Trailer Light Check Off
```
DELETE https://usapi.cv.ford.com/api/vehicles/VIN/trailerlightcheckactivation
```

## Queries
### Vehicles
```
GET https://services.cx.ford.com/api/dashboard/v1/users/vehicles
```
Returns
```javascript
Array<Vehicle>

typedef Vehicle {
    nickName: string,
    vin: string,
    vehicleType: string,
    color: string,
    modelName: string,
    modelCode: string,
    modelYear: string,
    tcuEnabled: integer, // looks to be boolean representation
    localMarketvalue: string,
    territoryDescription: string,
    vehicleAuthorizationStatus: {
        requestStatus: string,
        error: null | {
            statusContext: string,
            statusCode: integer, // HTTP code
            message: string, // stringified JSON
        },
        lastRequested: string, // Timestamp with timezone
        value: {
            authorization: string
        }
    },
    recallInfo: [
        {
            vin: string,
            numberOfRecalls: integer,
            recalls: UnknownType,
            requestsStatus: string,
            error: null | {
                statusContext: string,
                statusCode: integer, // HTTP code
                message: string, // stringified JSON
            },
            lastRequested: string, // Timestamp with timezone
        }
    ]
}
```

### SecuriAlert Status
```
GET https://api.mps.ford.com/api/guardmode/v1/VIN/session
```

### Vehicle Status
```
GET https://usapi.cv.ford.com/api/vehicles/v4/VIN/status
```
Returns
```javascript
{
    vehicle: {
        vin: string,
        nickName: string,
        vehicleType: string,
        color: string,
        modelName: string,
        modelCode: string,
        modelYear: string,
        tcuEnabled: number, // Probably boolean as int
        make: string,
        cylinders: number, // Guessed
        drivetrain: string, // Guessed
        engineDisp: string,
        fuelType: string, // "G" = gas
        series: string,
        productVariant: string, // Guessed
        averageMiles: string, // Guessed, probably number encoded as string
        estimatedMileage: string,
        mileage: string, // Guessed
        mileageDate: string, // Timestamp without timezone
        mileageSource: string, // Guessed
        drivingConditionId: integer, // -1 = invalid
        configurationId: integer, // Guessed
        primariyIndicator: string, // "" is invalid
        licenseplate: string,
        purchaseDate: string, // Guessed
        registrationDate: string, // Timezone with timestamp = purchase registration, not renewal
        ownerCycle: string, // Guessed
        ownerindicator: string,
        brandCode: string, // Ford or Lincoln?
        vehicleImageId: string, // Guessed
        headUnitType: string, // Guessed
        steeringWheelType: string, // Guessed, probably indicates heated or not
        lifeStyleXML: string, // probably XML, invalid is ""
        syncVehicleIndicator: string, // invalid is ""
        vhrReadyDate: string, // Timestamp without timezone
        vhrNotificationDate: string, // Timestamp without timezone
        vhrUrgentNotificationStatus: string, // Guessed
        vhrStatus: string, // Guessed
        vhrNotificationStatus: string, // Guessed
        ngSdnManaged: integer, // Probably boolean as int
        transmission: string,
        bodyStyle: string, // Guessed
        preferredDealer: string, // A code, "F" or "L" followed by 5 digits
        assignedDealer: string, // Guessed at same code as preferred
        sellingDealer: string, // Guessed at same code as preferred
        vhrReadyIndicator: string, // Guessed
        vehicleAuthorizationIndicator: integer, // Probably boolean as int
        hasAuthorizedUser: integer, // Probably boolean as int
        lastMileage: string, // Guessed
        vehicleRole: string, // Guessed
        warrantyStartDate: string, // Guessed at Timezone without timestamp
        versionDescription: string,
        vehicleUpdateDate: string, // Guessed at Timezone without timestamp
    },
    status: integer, // HTTP status code
    version: string, // Currently returns "1.0.0"
}
```

### Vehicle Information
```
GET https://usapi.cv.ford.com/api/users/vehicles/VIN/detail?lrdt=01-01-1970%2000%3A00%3A00
```

### Vehicle Capabilities
```
GET https://api.mps.ford.com/api/capability/v1/vehicles/VIN
```

### Vehicle OTA Information
_Locale and brand must be lower case._
```
POST https://www.digitalservices.ford.com/owner/api/v2/sync/firmware-update?vin=VIN&locale=LOCALE&brand=BRAND
```