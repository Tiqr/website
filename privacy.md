# Privacy Policy

tiqr is a general purpose solution for implementing strong authentication to a relying party (e.g. a web site). This privacy statement applies to the tiqr app, as published on the Google Play Store (android) and Apple Store (iOS). The relying party may have its own privacy policy, that is outside the scope of this statement.

## Personal Data
Account parameters includes a user identifier and a display name associated with the user. Both the user identifier and the display name can be arbitrary (random) strings and are typically determined by the relying party deploying tiqr for secure login.

As tiqr is a general purpose authentication solution, the exact nature of these account parameters are beyond the scope of this privacy statement. Please refer to the privacy statement of the relying party.

## Camera Access and Internet usage
Camera access is used only for scanning QR codes during enrollment of new tiqr accounts and optionally during authentication for communicationg authentication challenges from the server.

Internet access is used only for retrieving tiqr metadata containing account parameters and during authentication for communicating authentication responses to the server.

## Piggy Bank Relying party
For the “Piggy Bank” demo relying party (demo.tiqr.org), the user is asked to register a user identifier and a display name of his or her choosing. They can both be random but the user identifier must be unique. These account parameters are communicated over HTTPS (protected by TLS 1.2) and stored on the server’s file system.