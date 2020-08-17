---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - cURL

includes:
  # - errors

search: true
---

# Introduction

Welcome to the Garmin Connect Vantage Fit API Documentation.

# Authentication

> To authorize, use this code:

```shell
# pass the X-XSRF-TOKEN in header with each request
curl -X GET \ api_endpoint_here \
  -H 'X-XSRF-TOKEN: usertoken'
```

> Make sure to replace `usertoken` with your session token.

We expect for the X-XSRF-TOKEN  to be included in all API requests to the server in a header that looks like the following:

`X-XSRF-TOKEN: usertoken`

<aside class="notice">
You must replace <code>usertoken</code> with your personal API key.
</aside>

#Garmin Integration [Internal]

# OAuth Connect

OAuth 1.0a is implemented with Garmin Services to achieve authentication. Authentication is achieved in 3 steps, Redirection, callback and authentication

# 1. Redirection

## Generate the redirect and callback url.

```shell
curl -X GET \ https://api.vantagecircle.com/vantagefit/api/v1/garmin/redirecturl \
  -H 'X-XSRF-TOKEN: usertoken'
  -H 'Content-Type: application/json'
```

> The above API returns JSON structured like this:

```json
{
    "redirect_url": "https://connect.garmin.com/oauthConfirm?oauth_token=4e1e9c77-4db8-4d22-bca0-5e43df8eb184",
    "callback_url": "https://www.vantagefit.io/garmin-connect/"
}
```

This endpoint generates the redirection url as "redirect_url" with unauthenticated authorization token appended and the callback url as "callback_url". The callback url will be needed by the frontend interface to know when the redirection is complete.

### HTTP Request

`GET https://api.vantagecircle.com/vantagefit/api/v1/garmin/redirecturl`


### Response

Response contains a redirect_url property and a callback_url property.

Parameter | Type | Description
--------- | ------- | -----------
redirect_url | String | The URL to begin redirection to.
callback_url | String | The URL to observe end of redirection.

<aside class="success">
Remember — Make sure to store the callback url in session variable.
</aside>

# 2. Callback

## Redirect and Observe callback.

Upon receiving the redirection url , open it in a web view and once the user authenticates the request the webview will redirect to the callback url, observe for redirection to callback url and pick up the verify signature and request token from the URL to call the Authenticate API mentioned below to verify authentication status.

The final callback URL will be in the following structure

`https://www.vantagefit.io/garmin-connect?oauth_verifier=wceCIIMwYd&oauth_token=4e1e9c77-4db8-4d22-bca0-5e43df8eb184&user_id=657522`

Parameter | Type | Description
--------- | ------- | -----------
oauth_verifier | String | The verifier token to validate authentication.
oauth_token | String | The initial request token to validate authentication.
user_id | String | The user Identifier. optionally can be checked in frontend also to validate the token being received is for current user.

<aside class="success">
Note — Make sure to verify all three properties are present and not null or empty otherwise the callback is a failure.
</aside>

# 3. Authentication

## Validate Authenticate signature.

```shell
curl -X POST \ https://api.vantagecircle.com/vantagefit/api/v1/garmin/authenticate \
  -H 'X-XSRF-TOKEN: usertoken'
  -H 'Content-Type: application/json'
  -d '{
    "oauth_verifier": "M2HYTu09w7",
    "oauth_token":"52480e8d-a3f2-483a-b7fd-deb22f6692ce",
    "user_id":657522
    }'
```

> The above API expects JSON structured like this:

```json
{
    "oauth_verifier": "M2HYTu09w7",
    "oauth_token":"52480e8d-a3f2-483a-b7fd-deb22f6692ce",
    "user_id":657522
}
```

> and the response if successfull will have code 200

```json
{
    "status_message": "Garmin Connect Integration Successfull"
}
```

> and the response if failure will have code in 4xx series

```json
{
    "err": "Error message"
}
```

This endpoint validates the callback signature and generates an access token which is further saved against the user. This validation is the final step to connect Vantage Fit to Garmin Connect Web service.

### HTTP Request

`POST https://api.vantagecircle.com/vantagefit/api/v1/garmin/authenticate`

### Request Body

Request body must contain the following values received upon successful callback.

Parameter | Type | Description
--------- | ------- | -----------
oauth_verifier | String | The Verifier signature.
oauth_token | String | The unauthenticated request token.
user_id | String | The user ID as received.


### Response

Response will contain a success status message with code 200 or error message with code in 4xx series accordingly.

Parameter | Type | Description
--------- | ------- | -----------
status_message | String | The success or status message.
err | String | The error message.


<!-- # Nominating Employees

## Search for employees

```shell
curl -X GET \ https://api.vantagecircle.com/api/rewards/user/search?source=nominate&q=keyword&limit=10 \
  -H 'X-XSRF-TOKEN: usertoken'
```

> The above command returns JSON structured like this:

```json
{
  "results": [
    {
      "name": "Jon Doe",
      "id": "2436537",
      "email": "jon.doe@organization.com"
    }
  ]
}
```

This endpoint retrieves all employees matching the keyword.

### HTTP Request

`GET https://api.vantagecircle.com/api/rewards/user/search?source=nominate&q=keyword&limit=10`

### Query Parameters

Parameter | Type | Description
--------- | ------- | -----------
q | String | the search keyword (could by employee's email or name).
source | String | signifies the intent of search, acceptable values are 'appreciate' and 'nominate'.
limit | Int | limits the number of results to retrieve.

### Response

Response contains a list of search results, each result has the following data.

Parameter | Type | Description
--------- | ------- | -----------
name | String | The name of this search result.
id | Int | Unique user identifier of this search result.
email | String | The email address of this search result.

<aside class="success">
Remember — Make sure to change the keword in endpoint to your own search keyword.
</aside>

## Get Available Awards

```shell
curl -X POST \
  https://api.vantagecircle.com/api/award/config \
  -H 'X-XSRF-TOKEN: usertoken' \
  -d '{
      "receiverIds": [123456, 789102]
  }'
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "countryFilter": true,
    "groupFilter": true,
    "showBalanceBudget": true,
    "balanceBudget": "48500",
    "countries": [
      {
        "is_current_country": true,
        "country_id": 61,
        "country_name": "Australia",
        "country_flag": "https://res.cloudinary.com/du0mlu2n6/image/upload/FlagImages/australia.png",
        "currency_name": "AUD",
        "currency_hex": "&#x41;&#x24;",
        "pointsPerUnitCurrency": 100,
        "available_cities": [
          {
            "city_id": 19,
            "city": "Australia"
          }
        ]
      }
    ],
    "awards": [
      {
        "displayName": "ALL",
        "values": [
          {
            "rewardId": 52,
            "rewardName": "Standout Performer",
            "description": "",
            "points": 5,
            "image": "https://res.cloudinary.com/vantagecircle/image/upload/v1558071457/rewards_and_recognition/awards/vc/vc_standoutperformeraward.png",
            "background": "https://res.cloudinary.com/vantagecircle/image/upload/v1567151399/rewards_and_recognition/background/default_background-vantagecircle-5.png",
            "countryId": 91,
            "rewardCategory": [
              "Leadership",
              "Customer Excellence",
              "Collaborative"
            ],
            "isMultilevelApproverReward": false,
            "isAwardPointsEditable": true
          }
        ]
      },
      {
        "displayName": "MONTHLY",
        "values": [
          {
            "rewardId": 2,
            "rewardName": "MAKE A DIFFERENCE",
            "description": "Standout Performer",
            "points": 1000,
            "image": "https://res.cloudinary.com/vantagecircle/image/upload/v1557828925/rewards_and_recognition/awards/vc/vc_makeadifference.png",
            "background": "https://res.cloudinary.com/vantagecircle/image/upload/v1567151399/rewards_and_recognition/background/default_background-vantagecircle-5.png",
            "countryId": 91,
            "rewardCategory": [

            ],
            "isMultilevelApproverReward": false,
            "isAwardPointsEditable": true
          }
        ]
      }
    ]
  }
}
```

This endpoint provides available awards for the current country.

### HTTP Request

`POST https://api.vantagecircle.com/api/award/config`

### Request Parameters

Parameter | Type | Description
--------- | ------- | -----------
recieverIds | [Int] | list containing unique ids of employees to be nominated.

### HTTP Response

### Response

Parameter | Type | Description
--------- | ------- | -----------
countryFilter | Bool | Whether or not awards for other countries are available.
groupFilter | Bool | Are there multiple group of awards available in the current list.
showBalanceBudget | Bool | Whether or not to show the currently available budget.
balanceBudget | String | Current Nomination Budget for this award set and this country.
countries | [Country] | A list of countries available to select other awards.
awards | [AwardGroup] | A list of Award Groups available to select awards from.

### Country

Each country object has the following fields

Parameter | Type | Description
--------- | ------- | -----------
is_current_country | Bool | Whether this is the current country of nominator.
country_id | Int | Unique Country Identifier.
country_name | String | Country Name.
country_flag | String | Current Flag's image URL.
currency_name | String | Currency Name.
currency_hex | String | Currency Hex.

### AwardGroup

Each AwardGroup object has the following fields

Parameter | Type | Description
--------- | ------- | -----------
displayName | String | Group Name.
values | [Award] | List of Awards.

### Award

Each Award object has the following fields

Parameter | Type | Description
--------- | ------- | -----------
rewardId | Int | Award Identifier.
rewardName | String | Award Name.
description | String | Award Description.
points | Int | Award Points.
image | String | Award Logo.
background | String | Award's Background Image.
rewardCategory | [String] | List of Hashtags associated with this Award.
isMultilevelApproverReward | Bool | Whether the award is a Panel Award.
isAwardPointsEditable | Bool | Whether the award points are editable. -->

