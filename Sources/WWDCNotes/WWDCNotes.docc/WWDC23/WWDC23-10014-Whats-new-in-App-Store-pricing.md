# What’s new in App Store pricing

Discover the latest updates to App Store pricing capabilities and tools. Learn how you can manage pricing for your apps and in-app purchases within App Store Connect and the App Store Connect API, how to set pricing by region, and more.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10014", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Chong Chung and Claire Liu, App Store Connect Engineering

## Enhanced global pricing

- There are 44 currencies in 175 regions.
- There are now 900 price points (800 by default and 100 more upon request)

A base region for your base price can be configured from one of the 175 regions. The generated prices will be automatically adjusted to keep global prices in line with your base price as foreign exchange rate and tax fluctuate.

> For instance, if the UK pound and Japanese yen weaken against the US dollar and other currencies, prices in the UK and Japan would increase to maintain price equalization with the base US price.

If you set a custom price for a region, it won't be adjusted automatically according to the base price.

Automatic adjusted price changes will be shown in App Store Connect.

## Set up global pricing

Between 4:38 and 8:49, Claire guides you through how to set up initial pricing and schedule a global price change in App Store Connect.

### App Store Connect API

#### Getting info about app price schedule

> You can get your app's price schedule, including current prices and upcoming changes, from the AppPriceSchedules resource. AppPriceSchedule has three relationships: base territory, manual prices, and automatic prices.

> Manual Prices and automatic prices contain multiple app prices.

> Each AppPrice resource is linked to an AppPricePoint, which is linked to a territory. App price point and Territory are read-only reference data. So, I need to make three API calls to get the entire price schedule.

1. Get base country or region in app price schedule

`GET https://api.appstoreconnect.apple.com/v1/appPriceSchedules/<app-id>/baseTerritory`

```javascript
// Response
{
  "data" : {
    "type" : "territories"
    "id" : "USA",
    "attributes" : {
      "currency" : "USD"
    }
  }
}
```

2. Get manual prices in app price schedule

`GET https://api.appstoreconnect.apple.com/v1/appPriceSchedules/<app-id>/manualPrices?include=appPricePoint,territory`

```javascript
// Response
{
  "data" : [ {
    "type": "appPrices",
    "id" : "eyJzIjoiNjQOODQwMTY5NyIsInQiOiJVUOEiLCJwIjoiMTAxMDEiLCJZZCI6MC4wLCJ1ZCI6MC4wfQ"
    "attributes" : { "manual" : true, "startDate" : null, "endDate" : null },
    "relationships" : {
      "appPricePoint" : {
        "data": {
          "type" : "appPricePoints",
          "id" : "eyJzIjoiNjQOODQwMTY5NyIsInQiOiJVUOEiLCJwIjoiMTAxMDEifQ"
        }
      },
      "territory": {
        "data" : { "type" : "territories", "id" : "USA" }
      }
    }
  }]
}
```

3. Get automatic prices in app price schedule

`GET https://api.appstoreconnect.apple.com/v1/appPriceSchedules/<app-id>/automaticPrices?include=appPricePoint,territory`

```javascript
// Response
{
  "data" : [ {
    "type" : "appPrices",
    "id" : "eyJzIjoiNjQ0ODQwMTY5NyIsInQiOiJDQU4iLCJwIjoiMTAxMjciLCJzZCI6MC4WLCJ1ZCI6MC4wfQ"
    "attributes" : {
      "manual" : false,
      "startDate" : null,
      "endDate" : null
    },
    "relationships" : {
      "appPricePoint" : {
        "data" : {
          "type": "appPricePoints"
          "id' : "evJzIjoiNiQOODQwMTY5NvIsInQiOiJDQU4iLCJwIioiMTAxMicifQ'
        }
      }
    }
  },
  ...
  ]
}
```

#### Update price schedule

1. Get all price points of base country or region

`GET https://api.appstoreconnect.apple.com/v1/apps/6448401697/appPricePoints?filter[territory]=USA`

```javascript
// Response
{
  "data" : [ {
    "type" : "appPricePoints"
    "id" : "eyJzIjoiNjQOODQwMTY5NyIsInQi0iJVUEiLCJWIJoiMTAXMjcifQ",
    "attributes": {
      "customerPrice" : "9.99",
      "proceeds" : "7.0",
    },
  },
  // ... more price points
  ]
}
```

2. Get the comparable price points of base price

`GET https://api.appstoreconnect.apple.com/v3/appPricePoints/eyJzIjoiNjQOODQwMTY5NyIsInQi0iJVUOEiLCJwIjoiMTAxMjcifQ/equalizations?include=territory`

```javascript
// Response
{
  "data" : [ {
    "type" : "appPricePoints",
    "id" : "eyJzIjoiNjQOODQWMTY5NyIsInQi0iJDQU4iLCJwIJoiMTAXNDIifQ",
    "attributes" : {
      "customerPrice": "12.99",
      "proceeds": "9.09",
    },
    "relationships": {
      "territory": { "data": { "type": "territories", "id": "CAN" } }
    },
  },
  // ... 173 price points in other countries or regions
  ]
}
```

3. Update app price schedule

`POST https://api.appstoreconnect.apple.com/vl/appPriceSchedules`

```javascript
// Request body
{
  "data": {
    "type": "appPriceSchedules"
    "attributes": {},
    "relationships": {
      "app": {
        "data": { "type": "apps", "id": "6448401697" }
      },
      "baseTerritory": {
        "data": { "type": "territories", "id": "USA" }
      },
      "manualPrices": {
        "data": [
          { "type": "appPrices", "id": "${newprice-0}" },
          { "type": "appPrices", "id": "${newprice-1}" }
        ]
      }
    }
  },
  "included": [
    {
      "id": "${newprice-0}",
      "relationships": {
        "appPricePoint": {
          "data": {
            "type": "appPricePoints"
            "id': "eyJzIjoiNiQOODQWMTY5NvIsInQiOiJVUOEiLCJwIjoiMTAxMDEifQ"
          }
        }
      },
      "type": "appPrices",
      "attributes": {
        "startDate": null,
        "endDate": "2024-01-01"
      }
    },
    {
      "id": "${newprice-1}",
      "relationships": {
        "appPricePoint": {
          "data": {
            "type": "appPricePoints",
            "id": "eyJzIjoiNjQOODQwMTY5NyIsInQiOiJVUOEiLCJwIjoiMTAxMicifQ'
          }
        }
      },
      "type": "appPrices",
      "attributes": {
        "startDate": "2024-01-01",
        "endDate": null
      }
    }
  ]
}
```

> You can use any values you want for these temporary IDs, as long as they follow this syntax with a dollar sign and braces around the ID.

> Keep in mind that if a price's start date is null in the payload, the price will go live immediately in the associated region. So I recommend testing this API with an app that is not live in the App Store yet.

## Pricing by region (custom pricing)

When you set a custom price for a region, the price for that region won't automatically adjust.

### Temporary pricing

It is possible to set a custom price for a region with a start date and an end date.

### Custom pricing

It is possible to set a custom price for a region, so it won't automatically adjust. It is also possible to do this for the base region.

Between 18:58 and 22:30, Claire guides you through how to set up temporary prices and custom prices pricing in App Store Connect.

### App Store Connect API

Update app price schedule with a temporary price
 
`POST https://api.appstoreconnect.apple.com/v1/appPriceSchedules`

```javascript
// Request body
{
  "data": {
    "type": "appPriceSchedules"
    "attributes": {},
    "relationships": {
      "app": {...},
      "baseTerritory": {...}, // USA
      "manualPrices": {
        "data": [
          { "type": "appPrices", "id": "${newprice-0}" }, // base price in USA
          { "type": "appPrices", "id": "S{newprice-1}" } // temporary price in CAN
        ]
      }
    }
  }
  "included": [
    {
      "id": "${newprice-0}",
      "relationships": {
        "appPricePoint": {
          "data": {
            "type": "appPricePoints"
            "id": "eyJzIjoiN QOODQWMTY5NyIsInQi0iJVUOEiLCJwIjoiMTAXMDEifQ"
          }
        }
      },
      "type": "appPrices",
      "attributes": {
        "startDate": null,
        "endDate": null
      }
    },
    {
      "id": "${newprice-1}",
      "relationships": {
        "appPricePoint": {
          "data": {
            "type": "appPricePoints"
            "id": "eyJzIjoiNQODQMTY5NyIsInQiOiJDQU4iLCJWIjoiMTAWNJIifQ'
          }
        }
      },
      "type": "appPrices",
      "attributes": {
        "startDate": "2024-02-01",
        "endDate": "2024-02-03"
      }
    }
  ]
}
```

Get automatic prices in app price schedule

`GET https://api.appstoreconnect.apple.com/v1/appPriceSchedules/6448401697/automaticPrices?include=appPricePoint,territory`

```javascript
// Response
{
  "data" : [
    // automatic app price in Canada before sale
    { "attributes" : { "manual" : false, "startDate" : null, "endDate": "2024-02-01" } },
    // automatic app price in Canada after sale
    { "attributes" : { "manual" : false, "startDate" : "2024-02-03", "endDate" : null } },
    // ...
    // 173 automatic prices in other regions outside of Canada and the United States
    // { "attributes" : { "manual" : false, "startDate" : null, "endDate" : null } 
  ]
  "included" : [...]
}
```
