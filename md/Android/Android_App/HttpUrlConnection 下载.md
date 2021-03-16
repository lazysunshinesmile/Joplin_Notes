HttpUrlConnection 下载

HttpUrlConnection 下载

``` java
URL url = new URL(urlStr);
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
urlConnection.setRequestMethod("GET");
urlConnection.setDoOutput(false);

urlConnection.setConnectTimeout(10 * 1000);
urlConnection.setReadTimeout(10 * 1000);
urlConnection.setRequestProperty("Connection", "Keep-Alive");
String input = userName + ":" + password;
String encodingInput = Base64.encodeToString(input.getBytes(), 0);
//加密码
urlConnection.setRequestProperty("Authorization", "Basic " + encodingInput);

urlConnection.connect();
int statusCode = urlConnection.getResponseCode();
if (statusCode != HttpStatus.SC_OK) {
    urlConnection.disconnect();
    if (statusCode == 401) {
        throw new AuthenticatorException();
    }
}
inputStream = urlConnection.getInputStream();
```