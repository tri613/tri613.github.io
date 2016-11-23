---
title: Setup Google login / en
tags:
  - google
  - social-login
date: 2016-11-23 13:51:28
---

*Assume projects are already created in [Google API Console](https://console.developers.google.com/projectselector/apis/library).*
## 1. Setup with meta

```html
<!-- set client id in meta -->
<meta name="google-signin-client_id" content="your-client-id.apps.googleusercontent.com">

<!-- load google library -->
<script src="https://apis.google.com/js/platform.js" async defer></script>

<!-- set up login btn with classname `g-signin2` -->
<div id="google-login-btn" class="g-signin2" data-onsuccess="onSignIn"></div>
```

## 2. Setup with `gapi.auth2`

```html
<!-- load google library and specify callback function -->
<script src="https://apis.google.com/js/platform.js?onload=onloadCallback" async defer></script>

<!-- set up login btn (no classname needed, only ID) -->
<div id="google-login-btn"></div>
```

```js
//implement callback function
var auth2;
function onloadCallback() {
    gapi.load('auth2', function() {
        auth2 = gapi.auth2.init({
            client_id: 'your-client-id.apps.googleusercontent.com'
        });
        //attach event callback for clicking login btn
        auth2.attachClickHandler('google-login-btn', {}, onSuccess, onFailure);
    });
    //render the default style google btn
    gapi.signin2.render('google-login-btn');
}
function onSuccess (googleUser) {
  var profile = googleUser.getBasicProfile();
  console.log('ID: ' + profile.getId()); // Do not send to your backend! Use an ID token instead.
  console.log('Name: ' + profile.getName());
  console.log('Image URL: ' + profile.getImageUrl());
  console.log('Email: ' + profile.getEmail());
}

function onFailure(error) {
    console.log(error);
};

```

- Futher more, you can verify user through backend server.

```js
//modify the onSuccess function to get user token and send it to server
function onSuccess (googleUser) {
  var id_token = googleUser.getAuthResponse().id_token;
  sendUserTokenToServer(id_token);
}

function sendUserTokenToServer(token) {
  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/verify-user-token.php');
  xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
  xhr.onload = function() {
    console.log('Signed in as: ' + xhr.responseText);
  };
  xhr.send('token=' + token);
}

```

```php
//verify-user-token.php

$token = $_POST['token'];
$ch = curl_init();
curl_setopt_array($ch, array(
  CURLOPT_URL => 'https://www.googleapis.com/oauth2/v3/tokeninfo?id_token='.$token,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST => "GET",
    CURLOPT_SSL_VERIFYPEER => false,
    CURLOPT_SSL_VERIFYHOST => false
));
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response);
echo $data->name; //here's the user name!
```


## 3. Login with backend server

You can use backend server to login your user with google,
that means you only have to put a link to your login page on frontend only.
There are google api clients in different languages,
and I'm using [Google's api php client](https://github.com/google/google-api-php-client) here.

You need to prepare at least two pages for this:

- Login page
  Login page page should simply redirect to Google's login page.

```php
// signin-with-server.php

require_once __DIR__.'/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig(__DIR__.'/client_secret.json'); //file downloaded from Google API Console
$client->addScope('profile');
//set redirect uri
$client->setRedirectUri('http://' . $_SERVER['HTTP_HOST'] . '/callback.php');
$auth_url = $client->createAuthUrl();
header('Location: ' . filter_var($auth_url, FILTER_SANITIZE_URL));
exit();
```
> check all the scopes that you can use [here](https://developers.google.com/identity/protocols/googlescopes).

- Callback page
  After the user login on Google's login page, 
  Google will redirect to the uri that you set in `Login page` with `GET` paramater `code`
  *Make sure you have this redirect uri stored in the list of `Redirect URI` at Google API Console.*

```php
//callback.php

require_once __DIR__.'/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig(__DIR__.'/client_secret.json');
$client->addScope('profile');
$access_token = $client->fetchAccessTokenWithAuthCode($_GET['code']);

var_dump($access_token);
```