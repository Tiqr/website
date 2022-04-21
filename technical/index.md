# Technical

## So how does it really work?
Well, tiqr is based on Open Standards from the [Open Authentication Initiative](http://www.openauthentication.org/) (OATH). It uses the OCRA protocol suite to perform [challenge/response authentication](http://en.wikipedia.org/wiki/Challenge-response_authentication). Where traditional methods require you to type in (alpha-)numerical codes displayed on the web page, we leverage the ease-of-use of QR tags. And where you normally would have to copy the response from your phone by typing it on your computer we make use of the fact that almost all modern phones have Internet connectivity. This is the secret behind tiqr’s ease of use.

## Security
We use industry standard open cryptography in tiqr. All your identities are secured using 256-bit [AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) encryption. Furthermore, we are currently undergoing a rigorous security audit by a trusted external security auditor. Once the results are out, we will share them with you on this site.

## Data flow
The diagram below shows the data flow for a normal, successful authentication using tiqr:

![How](/img/tiqr-how-does-it-work.png)

The diagram shows the following steps:

1. The user surfs to a web site and is required to log in
2. The web site displays a QR code
3. The user scans the QR code using the tiqr App on his or her phone, confirms login and enters his/her PIN
4. The user’s identity together with the response to the challenge encoded in the QR tag is sent to the server using the phone’s Internet connection
5. The server validates the response and authorises login
6. The browser reloads the page and the user is logged in