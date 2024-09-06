# Tiqr Protocol

The tiqr-app and the tiqr server (website) need to communicate information during enrollment and authentication. 
## Enroll

During enrollment a user registers his Tiqr-app with the server

[![Flow diagram enrollment](./enroll.png)](https://sequencediagram.org/index.html#initialData=C4S2BsFMAIBUQI4CdqQHZIPbnAW3cAFCEAOAhkqAMYjlrADmWAridAMTggMAWwARuGYxIAE2YBJACKEAkKLLAy-MgGcYzdSlXBMSMg0hzylEDTrA4iJAFotAN0i2u-Dl16MkkdMYrVaZPTQAOqQ-A5O0ACqEhxeooTootDEJv4WVsgA9GKSUtBkJGyc3HxM3mjEoeFOjigxNgB88Mh2tU42LgBcAMpKlKgY2HgE0AAUmk4gogA00JNIoiCqJOBkAJ5oZPgAlIRomMAwmHWZthHOIPxdAOLoToqQXQA6aDaDWDj49AD6ANaQdbEFrndqXfhNBbQHR6AxPPp6DRaApoZIA9bQEBoaGQVSqECYSqEEFtJB1TpXJrVC7RCRdACiQy+owA0oDoAANYgHI7QE6Ralg2m3e76XnoT4jILMJDgV53NAPXkARQASjYqJhREZCGQqKB7I8QmEaTFdfqQIbeSSLhT+IRBWTIg0mjawXaugAFYbQpTATRVE1ChrNaw5cTSApFLrQHpUQKK5JRVUAGWBYdykcKJFd1lJ5O6N3psGg+CUCiUrz4wBIqi6WXDzGmADo0OAsqBkBLhlky2QK2QAPzd5m-dEAXi5xLztpcAB5IciYfpDF1wJhMH9WNAANRYw1cCsiJlS4D-QGvQyWPsDwM1J31CQ2XOtWdXL0+nSKAPcw7HU5ug+HoKkqx6St8Z7qFQXhENOr7ui4VJBg+woALKQOWihkK8ExaD80xzJg0FkD8qhNkcMwjhBpGQNBGF7A6yGnC6obwUB3Tejgvrfqod6mk+rFII2WbRuhmFKOM6gwPwkDrgA7gxPL-pEILCfk2YxiBYowAA8gAwqqACCOJ0bBqmZupRRIfezF0tWtb1o2LZth21gju27lZAAZliyw8DY7kQcOJ7UVBMHjgAmtA0CejpPQll5ei4AFaCatqogvGgYxSdAMnyQxjq2S+oLse+jKStAA44WsaAMDMPIgD58agISwDrCQkAzHU+KEjMCwEWFGGUSFBA0aZDGAQWVwLo0ULLnCXQAGpkIeRo7piaAHtMRqBaNg1mTOCGUrNS66Cu8JnTA+18cGT7FfmHQcZ+fo-sQxBAA)

### metadata_url

The metadata contains information about the tiqr service and the new user account this is being created. For security reasons the metadata_url must be uniqe so that a client can only retrieve the metadata once. The server invalidates the URL after providing the metadata so that it cannot be used a second time. A server can E.g. add a secure pseudorandom nonce to the URL to do this.

The format of the URL not specified. A client put extra restrictions on which servers it trusts.  

Example _metadata_url_: `https://tiqr.example.org/get_metadata/?enrollment_key=f835ae8f915433c11c9cded94280e3a7`  

### enrollment_url
The _erollemnt_url_ uses a custom URL scheme or universal link to start the enrollment process of a new user in the Tiqr client and to transfer the _metadata_url_ to the client. The _enrollment_url_ is transferred to the Tiqr client by the user scanning a QR code of by a clickable link.  

There are two formats defined for the enrollment URL

#### Custom URL Scheme
 
_scheme_://_metadata URL_  

E.g.: `tiqrenroll://https://tiqr.example.org/get_metadata/?enrollment_key=f835ae8f915433c11c9cded94280e3a7` where `tiqrenroll` is the cutorm url scheme that rigstered by the tiqr client.

#### Universal Link ####

_universal link_?metadata=_URL encoded metadata_url_  

E.g.: `metadata=https%3A%2F%2Ftiqr.example.org%2Fget_metadata%2F3Fkey%3Df835ae8f915433c11c9cded94280e3a7`  

### Metadata format

The tiqr client makes a HTTP GET request to the tiqr server. When the metadata URL is valid the server responds with an application/json encoded message.

- **service/displayName**: The friendly name of the tiqr server for display purposes
- **service/identifier**: A uniqe identifier of for this tiqr server
- **service/logoUrl**: A URL where the client can get a .png of .jpg image to associate with the account
- **service/infoUrl**: A URL where the user can get more information about the tiqr server
- **service/authenticationUrl**: The URL what the authentication response will be POSTed to when the user authenticates to the server
- **service/ocraSuite**: The OCRA suite to use, used to calculate responses. Not that the protocol currently does not support switching to a different suite.
- **service/enrollmentUrl**: The url where the client POSTs the secret. This URL must be uniqe and must be kept secret between the client and the server for the duration of the enrollment. This enrollment_key and the enrollment_secret must be different.

- **identity/identifier**: The identitfier of the user account. e.g. the login name, account name of the user at the server
- **identity/displayName**: The name of the user or some other description of the account.
```
{
    "service":{
      "displayName":"Tiqr example authentication server",
      "identifier":"tiqr.example.org",
      "logoUrl":"https:\/\/tiqr.example.org\/logo",
      "infoUrl":"https:\/\/tiqr.example.org\/info",
      "authenticationUrl":"https:\/\/tiqr.example.org\/authenticate",
      "ocraSuite":"OCRA-1:HOTP-SHA1-6:QH10-S064",
      "enrollmentUrl":"http:\/\/tiqr.example.org\/finish-enrollment?enrollment_secret=3cc68ea291fc5fbef20174d493fba41c"
   },
   "identity":{
      "identifier":"example-user",
      "displayName":"Example user"
   }
}
```

The app wil preform the enrollment, and will send the results to the `enrollmentUrl` using a `FORM POST`

With this information the tiqr client can create a user account in the client and generate the OCRA secret. This is a shared secret between the client and the server.

### Finish Enrollment

The tiqr client POSTs the FORM DATA to the server.

- **secret**: The ocra secret the the client generated. The server will store that and associate it with the userid. It is required to later authenticate the user.
- **language**: The preferred langue of the user
- **notificationAddress**: Optional. The device notification addess for receiving push notifications
- **notificationType**: Optional. The type of the address, this specifies the method that must be used to the notifiaction to the client.
- **operation**: Must always be `register`

Spported notification types:  
- **APNS**: Apple push notification service. The notificationAddress is a pseudotoken from a tiqr token exchange.
- **APNS_DIRECT**: Apple push notification service. The notificationAddress is the APNS device token.
- **FCM**: Firebase cloud messaging. The notificationAddress is a pseudotoken from a tiqr token exchange.
- **FCM_DIRECT**: Firebase cloud messaging. The notificationAddress is the FCM address.

Example:
```
secret=b57940c0939bd997628f36264409b29e9a5e10834fd227347698bb9146ae09a6
language=nl
notificationAddress=D5D760D233FC48194A546EB718917451FDC268E4E416A0AE87CEF77909F1EA81
notificationType=APNS_DIRECT
operation=register
```

If the enrollment information was processed corectly, the tiqr-server will return a `HTTP 200` message containing the string `OK`.

## Authentication

[![Flow diagram enrollment](./auth.png)](https://sequencediagram.org/index.html#initialData=C4S2BsFMAIBUQI4CdoEMCuwAWkB2oBjVUAe1wChyAHVJQkG-AcyRPSugGJwQmtgARuHQxIAE3QBJACLkAkGOKoBqAM4x06lKuAkkqJpGq16jYHERIAtDwFcefYC0h55NOiAINU+aAHVIAS0AN0gUAFVJLiRxcjwxaEp3Ux9zeGQAenEpaTQqDm5efmdXcnTrWwAeKwCgsNCIyQAuAGVgE2gAQUwcfE9iEDJoAApNMJAxAEpyXBJgGBIGi2QbEAEmgHE8MOJIJoAdXCtodVVVQdwAfQBrSABPQ+OCLFRwKFxDSnIx7V19Q2q5VW6zaehgp3OZBu92gAGpoM9Xu9PkDbFYAHy1EJhaCRJoQi7Qu6UWbzaCLHFY+o4vFbXA7MkYbB4QgDIboJDgElzBZLKlIJa07b6MkARQASlYCCQxJBDnpDi14tAqJosNBSSAAGb9UgUcj8wWSDGotZNAAKaugADk5trdRduWSKShTSDlarVOrNTqiHrKKgCKBgrt-IFsY1yIHg6G3Qbw9TGlYTZZgRaSG8Tu1gJoRgSobc7tN43UBTTjejylkJDI8lQmtAWkRcPSEuFxQAZDIkKh4FVWn0OshO3k4qvZWuofKbYWhgDyAGFxZ1oDFVFQyOpoFq9NBKoi3nhDOjKOOa7kp1QMYby01+MAqKomhlq+gJgA6XDgDKgZBMrAZP+LK6pAGSHOac4tLA256AAtlYeDSrKYgHLgwzqDAAiQOAJAAO7FjeSaVqmthNN0zJ9H6MCKO0hzoZAZwXAA0vcAA0PySGIrFrhuuDqKx4A+Ew6AGJArE9gyFysYcg5+hcwB3L2ABkVioGIa6qMWbrVOiPxZnookzuY6gEDE5g7igPwjuSSxuk0C6vAQ6CCWSi7LquDG8eohwLiQsHuDAuhdD0wFUdANGoF8boYnpOgGYYTThFQNHUUolCEbixopispHmhm4BZsQmjkLK0YgCGZJxqVQblaGGWRGUJFrNeCZlo0TQdiQTAgLgHmqM5wBfF8QA)

### authentication_url
The _authentication_url_ generated by the tiqr server and contains all information that the tiqr client requiers to start an authentication. This url can be presented to the user as a push-message, QR code or a clickable link.

The server sends:  
- **userid**: Optional. This is the **identity/identifier** that the server sent in the metadata during enrollment. Informs the tiqr client which user the server expects to authenticated
- **identifier**: This is the **service/identifier** that the server sent in the metadata during enrollment
- **session_key**: The OCRA SessionInformation (S) A secure pseudo random key that identifies this authentication
- **challenge_question**: The OCRA challenge question (Q) According to the **service/ocraSuite** that the server sent in the metadata during enrollment
- **server_identifier**: Identifier of the service to which the client is authenticating. Implementation defined. Set to **identity/identifier** when unknown
- **version**: The protocol version. Currenly 1 or 2.

There are two formats supported, depending on whether a custom URL scheme or a Universal Link is used.

#### Custom URL Scheme

With _userid_: _scheme_**://**_userid_**@**_identifier_**/**_session_key_**/**_challenge_question_**/**_server_identifier_**/**_version_  

Example: `tiqrauth://example-user@tiqr.example.org/0da1c51c3c3be54441527d4e5bde3710/747d558f3d/tiqr.example.org/2`  

Without _userid_: _scheme_**://**_identifier_**/**_session_key_**/**_challenge_question_**/**_server_identifier_**/**_version_  

Example: `tiqrauth://tiqr.example.org/0da1c51c3c3be54441527d4e5bde3710/747d558f3d/tiqr.example.org/2`  

#### Universal Link

With _userid_: _unversal Link_**?u=**_userid_**&i=**_identifier_**&s=**_session_key_**&q=**_challenge_question_**&v=**_version_  

Example: `https://tiqr.example.org/tiqrauth/?u=example-user&i=tiqr.example.org&s=0da1c51c3c3be54441527d4e5bde3710&q=747d558f3d&v=2`  

With _userid_: _unversal Link_**?i=**_identifier_**&s=**_session_key_**&q=**_challenge_question_**&v=**_version_  

Example: `https://tiqr.example.org/tiqrauth/?i=tiqr.example.org&s=0da1c51c3c3be54441527d4e5bde3710&q=747d558f3d&v=2`  

Note that all URL paramater values must be URL encoded and that this format does not support specifying a _server_identifier_


### POST /authenticate FORM DATA

The client calculates the OCRA response using the information from the authentication_url, the OCRA suit it received in the metadata and the OCRA client secret that it generated during enrollment.

It sends a POST with FORM DATA back using with:  
- **sessionKey**: the _session_key_ from the _authentication_url_
- **userId**: the usedid of the user that it authenticated
- **response**: the OCRA response that it calculated
- **language**: the preffered language of the user
- **operation**: always set to `login`
- **notificationType**: Optional. See enrollment.
- **notificationAddress**: Optional. See enrollment.

Example:
```
sessionKey=0da1c51c3c3be54441527d4e5bde3710
userId=example-user
response=012345
language=nl
operation=login
notificationType: APNS_DIRECT
notificationAddress: D5D760D233FC48194A546EB718917451FDC268E4E416A0AE87CEF77909F1EA81
```

If the authentication is processed corectly, the tiqr-server will return a `HTTP 200` message containing the string `OK`.
