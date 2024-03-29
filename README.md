android-sdk-webview 
===================
A simple application that uses the android webview sdk to integrate the connect flow within an app

How to add the dependency to your project
----------------------------------------- 
You need to add the aar to you project. If you are using gradle you can take an example at this app
structure to see how the dependency has been handled.

###In details: 

There are 2 aar in the mubi-webviewsdk module in this project:
* webviewsdk-prod-release.aar
* webviewsdk-prod-debug.aar (contains extra logs that the release don't record)

#####The new module configuration
Simply add either of those aar in a new aar gradle module. Let's call it 'mubi-webviewsdk'.
Make sure the file build.gradle of your new module contains the following:
```groovy
configurations.maybeCreate("default")
artifacts.add("default", file('webviewsdk-prod-release.aar'))
```
You can change to the debug aar if you want to see some logs of what is happening.

#####You main app module configuration
Make sure the file build.gradle of your main app module contains the following:
```groovy
 compile project(path: ':mubi-webviewsdk')
``` 

How does the webview SDK work
-----------------------------

* A Fragment that wraps a webview that will display a connect page and listen to both mubi schemes: mubiConnectFragment.mubi_URI_SCHEME
and mubiConnectFragment.mubi_URI_WEBSITE_SCHEME to decide trigger the mubi app via the 'connect button' of the mubi page. The mubi app
will then, according to the user decision, carry out the requested operation or not.
* In case of success, if one wants his mobile app to be invoked when the user has successfully carried out the mubi request within the mubi app,
he should make sure that at least one android component is declared in the manifest to listens to the callback defined when the
validation scenario has been created in mubi Dashboard.
* An example in case of the mubi Connect Page where the callback is constructed this way : "https://www.mubi.com/connect/thankyou/?token=yourTokenValueHere"
The manifest entry for an android component will have the following intent filter :
```html<pre>
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data
        android:pathPrefix="/connect/thankyou/"
        android:host="www.mubi.com"
        android:scheme="https"/>
</intent-filter>
</pre>
```
* Note : if the user denies the request within mubi app, he will to quit mubi app manually and no notification will be sent to the calling app.
In order for the host activity to be notified of all the possible outcome, it must implement mubiConnectFragment.OnmubiConnectWebViewListener

How to use the webview SDK
--------------------------
### The ```mubiConnectFragment```
* The fragment only requires the URL to be submitted at creation time. This URL should be the mubi connect
page that you want to display to your user. (the exact same one that will display a QRCode if opened on a desktop browser)
* This fragment exposes a interface that the host activity must implement ```OnmubiConnectWebViewListener```.
This interface callbacks will be invoked when the submitted URL is invalid or when no mubi app has been found to resolve
the mubi requests.

Here is how you instantiate the fragment:
```java
mubiConnectFragment fragment = mubiConnectFragment.newInstance(url);

// url is the URL pointing at the connect page you want your user to use

getFragmentManager()
        .beginTransaction()
        .add(R.id.container, fragment, "mubiFragment")
        .commit();
```
### The ```mubiConnectActivity```
An activity that wraps a mubiConnectFragment that will display a connect page.
In order for the caller to be notified of all the possible outcomes, it must start this activity for result
via Activity.startActivityForResult(Intent, int). One intent parameter is required and it is the URL needed by the
fragment to display the page. This will be passed with the key ```mubiConnectActivity.RGS_CONNECT_PAGE_URL```
Here the possible result returned by this activity:
* If none of those scheme can be resolved, then ```mubiConnectActivity.RESULT_NO_mubi_APP_FOUND``` will be returned</li>
* if no valid URL is submitted in the intent then ```mubiConnectActivityRESULT_NOT_VALID_URL_SUBMITTED``` will be returned</li>