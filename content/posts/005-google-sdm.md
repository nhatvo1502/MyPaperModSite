---
title: 'Using Google SDM API to control Google Nest Camera'
date: 2024-01-18T11:55:37-08:00
draft: true
author: Nhat Vo
---

Project name: Croddy 
- Nest Camera
```
Guide: https://developers.google.com/nest/device-access/get-started
```

1. Register Device Access
- Accept $5 one time fee
```
https://console.nest.google.com/device-access/
```

2. Activate Supported Device
- Activate and setup device with non-commercial google account

3. Setup Google Cloud Platform
- Use the link on top of the article, scroll down to Setup Google Cloud Platform and click on Enable the API and Get an OAuth 2.0 Client ID
- Select Webserver > Enter *https://www.google.com* > copy Client ID and download the file

4. Create a Device Access project
- Go to Device Access > Create new project > paste Client ID > disable event
- Once done, copy Project ID down

Now you have to link your developer account with user account

5. Link your account PCM - Get Authorization Code
```
https://nestservices.google.com/partnerconnections/project-id/auth?redirect_uri=https://www.google.com&access_type=offline&prompt=consent&client_id=oauth2-client-id&response_type=code&scope=https://www.googleapis.com/auth/sdm.service
```
- Go to Google Cloud Platform > OAuth consent screen > Under Test User > add yourself in
- replace project-id (from Device Access Project ID)
- replace oath2-client-id with OAuthClient ID from Google Cloud Crendetial
- Copy and paste to google
- Allow and follow next
- Once done, it will go to www.google.com, copy the URL
- The Authorization code is in the URL: 
```
https://www.google.com/?code=4/0AfJohXlvngYWOp06a-e68hfU7jEOyrMeUyPlA2AVfs-og4i172VeEBzLLgP7UsiWxGkXrA&scope=https://www.googleapis.com/auth/sdm.service
```

- the code start after code= and end &scope

6. Get Access Token
- Open terminal, replace
	- oauth2-client-id with the OAuth2 Client ID from your Google Cloud Credentials
	- oauth2-client-secret with  Client Secret from your Google Cloud Credentials
	- authorization-code with the code you received in the previous stepvariable and run 
```
curl -L -X POST 'https://www.googleapis.com/oauth2/v4/token?client_id=oauth2-client-id&client_secret=oauth2-client-secret&code=authorization-code&grant_type=authorization_code&redirect_uri=https://www.google.com'
```

- if it give you error, run
```
Remove-item alias:curl
```

- if it gives you Error 411 (Length Required), add this to the end of your curl
```
-H 'Content-Length: 0'
```

- it should return 2 tokens: access token and refresh token
```
{
  "access_token": "access-token",
  "expires_in": 3599,
  "refresh_token": "refresh-token",
  "scope": "https://www.googleapis.com/auth/sdm.service",
  "token_type": "Bearer"
}
```

7. Make a device list call
IMPORTANT: This call is required to complete the authorization, with out this call, the authorization project will not appear in PCM and events will not be published
```
curl -X GET 'https://smartdevicemanagement.googleapis.com/v1/enterprises/project-id/devices' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer access-token'
```
- Replace *project-id* with Project ID find in Deviec Access console
- Replace *access-token* with Access Token from step 6
- remove space and \ to make the curl into one line
- copy and paste into terminal
- result
```
{
  "devices": [
    {
      "name": "enterprises/project-id/devices/device-id",
      "type": "sdm.devices.types.device-type",
      "traits": { ... },
      "parentRelations": [
        {
          "parent": "enterprises/project-id/structures/structure-id/rooms/room-id",
          "displayName": "device-room-name"
        }
      ]
    }
  ]
}
```
- 