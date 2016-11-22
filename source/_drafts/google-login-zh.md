---
title: Setup Google login
tags: 
- google
- social-login
---

*假設已經在 [Google API Console](https://console.developers.google.com/projectselector/apis/library) 完成專案的建置並取得`Client Id`了*
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

- 在可以取得使用者的token讓後端去驗證使用者身分，並進行一些登入手續處理。（像是寫Session幹嘛的）

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

//記得先驗證 $data->aud 是否和client-id相同
if ($data->aud == MY_CLIENT_ID) {
  // $data->sub 則是使用者的 unique google id
  echo $data->name; //here's the user name!
} else {
  echo 'User not valid.';
}
```

## 3. Login with backend server (導頁)

也可以選擇直接用後端和Google溝通，
這樣就不需要載入Google的JS函式庫或寫任何的js script，
只需要在前端頁面加上連到登入導頁的連結即可。

Google本身提供了許多語言的sdk，
這邊我使用的是 [Google's api php client](https://github.com/google/google-api-php-client)。

實作這個功能至少需要兩個頁面(吧)：

- Redirect to Google page
  這個Redirect to Google page只需要單純實現導頁至`Google登入頁面`即可。

```php
// signin-with-server.php

require_once __DIR__.'/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig(__DIR__.'/client_secret.json'); //這個json檔可以從Google API Console下載取得
$client->addScope('profile');
//set redirect uri
$client->setRedirectUri('http://' . $_SERVER['HTTP_HOST'] . '/callback.php');
$auth_url = $client->createAuthUrl();
header('Location: ' . filter_var($auth_url, FILTER_SANITIZE_URL));
exit();
```

> check all the scopes that you can use [here](https://developers.google.com/identity/protocols/googlescopes).

- Google Callback page
  使用者在Google登入頁面上登入完畢後，Google會導回當初在前面設定的Redirect URI，並附上GET參數`code`。
  *Redirect URI必須存在於Google API Console的設定裡面。*

```php
//callback.php

require_once __DIR__.'/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig(__DIR__.'/client_secret.json');
$client->addScope('profile');
$access_token = $client->fetchAccessTokenWithAuthCode($_GET['code']);

var_dump($access_token);
```

## 參考文件

- [Google Sign-In for Websites](https://developers.google.com/identity/sign-in/web/)
  其實方法大致上都可以在官方文件中找到，上面有些範例也是從官方來的，
  只是我覺得官方文件不夠友善，有些東西都只說一半還得自己另外Google。(XD)

- [OAuth 2.0 Scopes for Google APIs](https://developers.google.com/identity/protocols/googlescopes)
- [Google API Client Libraries](https://developers.google.com/api-client-library/)