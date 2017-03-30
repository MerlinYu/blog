#android签名
转载博客：https://developer.android.com/studio/publish/app-signing.html
##签名
You do not need Android Studio to sign your app. You can sign your app from the command line using standard tools from the Android SDK and the JDK. 
To sign an app in release mode from the command line:<br>

1. Generate a private key using keytool.
$ keytool -genkey -v -keystore my-release-key.keystore
-alias alias_name -keyalg RSA -keysize 2048 -validity 10000
This example prompts you for passwords for the keystore and key, and to provide the Distinguished Name fields for your key. 
It then generates the keystore as a file called my-release-key.keystore. The keystore contains a single key, valid for 10000 days. 
The alias is a name that you will use later when signing your app.
2. Compile your app in release mode to ls an unsigned APK.<br>
3. Sign your app with your private key using jarsigner:<br>
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1<br>
-keystore my-release-key.keystore my_application.apk alias_name<br>
This example prompts you for passwords for the keystore and key. It then modifies the APK in-place to sign it.
Note that you can sign an APK multiple times with different keys.
4. Verify that your APK is signed. For example:<br>
$ jarsigner -verify -verbose -certs my_application.apk

##查看签名
1. 查看 keystore文件的签名信息<br>
 keytool -list -v -keystore keystoreName -storepass keystorePassword
2. 检查apk文件中的签名信息<br>
解出apk中RSA文件，然后用keytool即可查看签名信息：<br>
keytool -printcert -file ~/test/CERT.RSA
3. 查看APK签名信息<br>
keytool -printcert -file META-INF/CERT.RSA
