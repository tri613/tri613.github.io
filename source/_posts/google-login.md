---
title: Setup Google login
tags:
  - google
  - social-login
date: 2016-11-23 13:53:46
---


假設已經在 [Google API Console](https://console.developers.google.com/projectselector/apis/library) 完成專案的建置並取得`Client Id`了，
有三種方式可以設置Google登入。

## 1. Setup with meta

```html
<!-- set client id in meta -->
<meta name="google-signin-client_id" content="your-client-id.apps.googleusercontent.com">

<!-- load google library -->
<script src="https://apis.google.com/js/platform.js" async defer></script>

<!-- set up login btn with classname `g-signin2` -->
<div id="google-login-btn" class="g-signin2" data-onsuccess="onSuccess"></div>
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

- 在這可以取得使用者的token讓後端去驗證使用者身分，並進行一些登入手續處理。（像是寫Session幹嘛的）

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
//signin.php

require_once '/your/path/to/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig('/your/path/to/client_secret.json'); //這個json檔可以從Google API Console下載取得
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

> 一開始我一直收到 `redirect_uri_mismatch` 的錯誤，不知道是不是我在同一個憑證下設了兩個redirect_uri的關係，
反正我後來直接重開一個新的憑證就好了。(囧)

```php
//callback.php

require_once '/your/path/to/vendor/autoload.php';

$client = new Google_Client();
$client->setAuthConfig('/your/path/to/client_secret.json');
$client->addScope('profile');

$client->authenticate($_GET['code']);
$token = $client->getAccessToken();

$token_data = $client->verifyIdToken();
```

```php
//token
Array
(
    [access_token] => 
    [token_type] => 
    [expires_in] => 
    [id_token] => 
    [created] => 
)

//token_data
Array
(
    [iss] => 
    [exp] => 
    [at_hash] => 
    [aud] => //檢查這個欄位是否和專案的client_id相同
    [sub] => //uid
    [email_verified] => 
    [azp] => 
    [email] => 
    [name] => 
    [picture] => 
    [given_name] => 
    [family_name] => 
    [locale] => 
)
```
然後就可以透過 `$token_data` 取得使用者資料和他的uid(`sub`)了。

`$client->verifyIdToken()` 和上面提到 `verify-user-token.php`的功能是一樣的，
只是這邊是用SDK直接問，上面則是用`curl`的方式去問 google api 而已。

## 完整範例
  這是我自己測試時寫的程式碼，我把它放在[github](https://github.com/tri613/google-login)上。

## 參考文件

- [Google Sign-In for Websites](https://developers.google.com/identity/sign-in/web/)
  其實方法大致上都可以在官方文件中找到，上面有些範例也是從官方來的，
  只是我覺得官方文件不夠友善，有些東西都只說一半還得自己另外google。(囧)

- [OAuth 2.0 Scopes for Google APIs](https://developers.google.com/identity/protocols/googlescopes)
- [Google API Client Libraries](https://developers.google.com/api-client-library/)