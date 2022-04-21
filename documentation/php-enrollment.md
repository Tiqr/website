# Tiqr Library Integration Guide – Enrollment

In [the first part of the integration guide](php-authentication.md) we covered login integration. Before a user can login they have to create an account though and link it to their phone. This process is called enrollment. In the demo application this is simply a matter of clicking the ‘create an account’ link which allows anybody to create an account.

In your own applications the process may be quite different: you may have a corporate application and accounts may only be created by registered employees, so they login using their regular employee login first.

Tiqr allows you to design and implement your own enrollment scheme. For tiqr all that matters is that somewhere in the process an enrollment QR code is displayed which contains the user’s identity. This guide covers how you can integrate the tiqr library into your processes to perform this enrollment. How you get the QR code to your user (on a page, by email) is out of scope here – for the sake of simplicity we’ll assume that you’re displaying the QR code on a page similar to the demo environment.

The following diagram explains the enrollment process:

![enrollment-integration-guide.png](/img/enrollment-integration-guide.png)

Similar to the login integration, we are dealing with a number of ‘entrypoints’ here that you should implement. They form the glue between the tiqr library and the user’s browser and phone. Let’s look at each of them.

1. Newuser.php

This is the page that displays the QR code to the user. It has no interactions with tiqr, as it’s simply a wrapper to display the QR code. We have kept this deliberately simple since the QR code may end up being displayed on something other than a page (e.g. a letter), so most of the logic happens when the QR code is generated.

To give some visual feedback once the enrollment process is complete, we start an ajax request loop calling verify_enroll.php periodically. This way we can display a ‘you are now enrolled’ result page once the phone has completed the enrollment process.

2. Qr_enroll.php

This script generates the QR code that contains the user’s identity (actually, it just contains a temporary reference, the actual identity is retrieved later on in the process in a client/server protocol). Before it can generate the QR code it needs to call a startEnrollmentSession method on the Tiqr_Service object. startEnrollmentSession generates a key that will be used later in the process to secure the enrollment process. Also, startEnrollmentSession starts the clock; if the enrollment isn’t completed within 5 minutes after startEnrollmentSession is called, sensitive data is removed and the account is closed for enrollment. (This is to ensure that a QR code that gets sniffed, copied or stolen will be harmless after a few minutes.)

After calling startEnrollmentSession the script should call generateEnrollmentQR which outputs the QR code. As part of the call to generateEnrollmentQR you should pass the full URL to the metadata url that the phone will use to retrieve enrollment information. You should add the key that startEnrollmentSession returned to the URL, e.g. metadata.php?key=<thekey>.

3. Verify_enroll.php

This is a simple script that verifies the status of the current enrollment session. It calls Tiqr_Service’s getEnrollmentStatus method. If enrollment has completed (see the getEnrollmentStatus documentation for its return codes) it can redirect to complete_enroll.php to finalize the enrollment.

4. Metadata.php

This script is an important step in the enrollment process. It is called by the phone to retrieve important information, such as the URL where authentication responses should be posted, the userid, the name and logo of your application and various other bits. In your metadata.php script you must do a number of things:

First, you must exchange the $key that the phone passed to your script (which is the key generated in the qr_enroll.php script above) for a new, unique enrollment secret by calling getEnrollmentSecret on the Tiqr_Service instance. This is a one time password that the phone is going to use later to post the shared secret of the user account on the phone.
Next you add the enrollment secret as ?key= parameter to the url of your application’s enrollment script (enroll.php, see further down in this guide). This is your enrollment url which you use in the next step.
Next, you must pass the original $key, your application’s authentication url (e.g. post.php from the login integration guide) and your application’s enrollment url (from the previous bullet) to the getEnrollmentMetadata method of Tiqr_Service. This method will compile a package of metadata and return it as an associative array. Note that for security reasons you can only ever call getEnrollmentMetadata once in an enrollment session, the data is destroyed after your first call.
Finally, metadata.php is responsible for JSON encoding the metadata that the library returned and outputting it – this will send the metadata to the phone.
After reading the metadata, the phone knows everything about the user, about the application the user is enrolling for, and it has all the urls it needs to complete enrollment and to later perform authentications.

5. Enroll.php

After processing the metadata, the phone will generate a secure secret for the user account. This ties the user’s identity to the phone he is using for the enrollment. The phone needs to communicate this secret to your application, which is done by calling your application’s enroll.php script.

To ensure that only the user’s phone can call enroll.php and store the shared secret, the phone will send the enrollment secret that was generated in step 4 along with it. You should call Tiqr_Service’s validateEnrollmentSecret to verify if the secret is valid. If so, this method will return the userid of the user that was performing the enrollment. Note that the phone will not let you know what user is enrolling, as sending both userid and shared secret would be insecure. It will only send you the shared secret and the one-time enrollment secret. The server knows the userid that belongs to this one time secret and can now store the shared secret.

Tiqr does not store user data, this is your application’s responsibility. You should store the secret you have received from the phone in a secure location. Later you’ll use it for the login process.

Finally, your  enroll.php script should call finalizeEnrollment to let tiqr know that you’ve taken care of storing the user’s shared secret. Tiqr will now close the enrollment session.

6. Complete_enroll.php

Once the phone has completed enrollment, the browser will notice that enrollment has completed and redirect the user to complete_enroll.php. This can be used to give some feedback to the user such as ‘your account is succesfully enrolled’.

If your application is used to enroll multiple accounts, you should call Tiqr_Service’s resetEnrollmentSession method, which tells tiqr that another enrollment may now start. (If you don’t call this, tiqr will remain in ‘enrollment completed’ state, which is fine for normal setups, but not for scripts that need to enroll more than one user in the same session).