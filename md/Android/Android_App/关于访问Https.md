关于访问Https

## HttpUrlConnection跳过证书校验
```
public void requestWithoutCA() {
	try {

		SSLContext sc = SSLContext.getInstance("TLS");
		sc.init(null, new TrustManager[] { new MyTrustManager() },
				new SecureRandom());
		HttpsURLConnection
				.setDefaultSSLSocketFactory(sc.getSocketFactory());
		HttpsURLConnection
				.setDefaultHostnameVerifier(new MyHostnameVerifier());

		URL url = new URL("https://certs.cac.washington.edu/CAtest/");
		HttpURLConnection urlConnection = (HttpURLConnection) url
				.openConnection();

		InputStream in = urlConnection.getInputStream();
		// 取得输入流，并使用Reader读取
		BufferedReader reader = new BufferedReader(
				new InputStreamReader(in));
		System.out.println("=============================");
		System.out.println("Contents of get request");
		System.out.println("=============================");
		String lines;
		while ((lines = reader.readLine()) != null) {
			System.out.println(lines);
		}
		reader.close();
		// 断开连接
		urlConnection.disconnect();
		System.out.println("=============================");
		System.out.println("Contents of get request ends");
		System.out.println("=============================");
	} catch (MalformedURLException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (NoSuchAlgorithmException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyManagementException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}


private class MyHostnameVerifier implements HostnameVerifier {
	@Override
	public boolean verify(String hostname, SSLSession session) {
		// TODO Auto-generated method stub
		return true;
	}

}

private class MyTrustManager implements X509TrustManager {
	@Override
	public void checkClientTrusted(X509Certificate[] chain, String authType)
			throws CertificateException {
		// TODO Auto-generated method stub
	}

	@Override
	public void checkServerTrusted(X509Certificate[] chain, String authType)

	throws CertificateException {
		// TODO Auto-generated method stub
	}

	@Override
	public X509Certificate[] getAcceptedIssuers() {
		// TODO Auto-generated method stub
		return null;
	}

}
```

## 使用证书：
[参考](https://developer.android.com/training/articles/security-ssl.html)
该代码时访问华盛顿大学的一个网页，证书可以到以下链接下载：
https://www.washington.edu/itconnect/security/ca/load-der.crt
**注意，此处使用的是HttpsURLConnection，而非HttpURLConnection。**
下面的代码，加载证书等等操作，最终都是为了setSSLSocketFactory这一步
```
public void requestByHttps() {
	try {

		// Load CAs from an InputStream
		// (could be from a resource or ByteArrayInputStream or ...)

		CertificateFactory cf = CertificateFactory.getInstance("X.509");
		// From
		// https://www.washington.edu/itconnect/security/ca/load-der.crt
		InputStream caInput = this.getAssets().open("load_der.crt");
		Certificate ca;
		try {
			ca = cf.generateCertificate(caInput);
			System.out.println("ca="
					+ ((X509Certificate) ca).getSubjectDN());
		} finally {
			caInput.close();
		}

		// Create a KeyStore containing our trusted CAs
		String keyStoreType = KeyStore.getDefaultType();
		KeyStore keyStore = KeyStore.getInstance(keyStoreType);
		keyStore.load(null, null);
		keyStore.setCertificateEntry("ca", ca);

		// Create a TrustManager that trusts the CAs in our KeyStore
		String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
		TrustManagerFactory tmf = TrustManagerFactory
				.getInstance(tmfAlgorithm);
		tmf.init(keyStore);

		// Create an SSLContext that uses our TrustManager
		SSLContext context = SSLContext.getInstance("TLS");
		context.init(null, tmf.getTrustManagers(), null);

		URL url = new URL("https://certs.cac.washington.edu/CAtest/");
		HttpsURLConnection urlConnection = (HttpsURLConnection) url
				.openConnection();

		urlConnection.setSSLSocketFactory(context.getSocketFactory());
		InputStream in = urlConnection.getInputStream();
		// 取得输入流，并使用Reader读取
		BufferedReader reader = new BufferedReader(
				new InputStreamReader(in));
		System.out.println("=============================");
		System.out.println("Contents of get request");
		System.out.println("=============================");
		String lines;
		while ((lines = reader.readLine()) != null) {
			System.out.println(lines);
			// tv.setText(tv.getText().toString() + lines);
		}
		reader.close();
		// 断开连接
		urlConnection.disconnect();
		System.out.println("=============================");
		System.out.println("Contents of get request ends");
		System.out.println("=============================");
	} catch (MalformedURLException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (NoSuchAlgorithmException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyManagementException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (KeyStoreException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (CertificateException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}
```


