# Quasar-build-to-apk

#
### Команды для подписания апк (ключ)
После команды 

```batch
cordova build android --release -- --packageType=apk
```
у нас сгенерируется неподписанный apk, нужно будет его подписать.
Сначала сгенерируем ключ:

```batch
keytool -genkey -v -keystore [keystore_name].keystore -alias [alias_name] -keyalg RSA -keysize 2048 -validity 10000
```
Далее экспортируем сертификат

```batch
keytool -exportcert -rfc -alias [alias_name] -keystore /home/noort/Documents/quasar-project/src-cordova/[keystore_name].keystore -file output_upload_certificate.pem
```

Подписываем:

```batch
apksigner sign --ks /home/noort/Documents/quasar-project/src-cordova/[keystore_name].keystore --ks-key-alias [alias_name] --out /home/noort/Documents/quasar-project/src-cordova/platforms/android/app/build/outputs/apk/release/app-release-signed.apk /home/noort/Documents/quasar-project/src-cordova/platforms/android/app/build/outputs/apk/release/app-release-unsigned.apk
```

И выполняем проверку подписи APK


```batch
apksigner verify /home/noort/Documents/quasar-project/src-cordova/platforms/android/app/build/outputs/apk/release/app-release-signed.apk
```



#
## For Windows (скоро и на линукс)

### Run the following commands in the root folder of your app


```batch
npm install -g cordova
```

```batch
quasar mode add cordova
```

```batch
cd src-cordova
```

```batch
cordova platform add android
```

```batch
cd..
```

```batch
quasar build -m cordova -T android
```

```batch
cd src-cordova
```

```batch
cordova build android
```

Дальше берём и скачиваем Android studio чтобы не докачивать отдельно Android SDK

После чего открываем проект в андройд студио и пытаемся выполнить следущую команду:

```batch
quasar build -m android -- "--" --packageType=bundle
```

На что ide говорит нам:

"Checking Java JDK and Android SDK versions
ANDROID_HOME=undefined (recommended setting)
ANDROID_SDK_ROOT=undefined (DEPRECATED)
Using Android SDK: ______________
Could not find an installed version of Gradle either in Android Studio,
or on your system to install the gradle wrapper. Please include gradle
in your path, or install Android Studio
"

Дальше ставим Java JDK (в основном нас интересует как раз эта переменная тк мы скачали Android studio и не надо докачивать отдельно андроид сдк, как делают в прикрепленном ниже гайде) и переопределяем переменные для путей
так как без этого никуда (https://www.tutorials24x7.com/android/how-to-install-android-sdk-tools-on-windows)

Не забываем юзать -v для проверки установки а также переменные можно задать в консоли используя например
```bash
setx PATH "%PATH%;%GRADLE_HOME%\bin"
```

Мои итоговые переменные (как пример):

```bash
ANDROID_HOME = C:\Users\SystemX\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT = C:\Users\SystemX\AppData\Local\Android\Sdk
JAVA_HOME = C:\Program Files\Java\jdk-23
PATH = %USERPROFILE%\AppData\Local\Microsoft\WindowsApps;C:\Users\SystemX\AppData\Local\Android\Sdk\platform-tools;C:\Users\SystemX\AppData\Local\Android\Sdk\tools
```


После этого нужно будет скачать Gradle через cmd от имени админа (тк без него у меня просто не работает, хотя вроде бы в Android Studio gradle есть по дефолту)
Но поскольку так просто его не скачать - придется сначала поставить choco


```batch
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```
```batch
choco install gradle
```

```batch
gradle -v
```

После этого заходим в терминал андроид студио и вводим такие команды.
Это нужно потому что переменные окружения установлены неправильно для текущей сессии.

 ```batch
$env:GRADLE_HOME = "C:\Gradle\gradle-8.10.2"
$env:PATH += ";$env:GRADLE_HOME\bin"
```
 ```batch
gradle -v
```

После того как версия показалась пересоздаем android:

```bash
cordova platform rm android
cordova platform add android
```

Дальше билдим командой:

```bash
 quasar build -m cordova -T android -- --debug --packageType=apk
```

И апк должен был сгенерироваться по указанному пути, например:  C:\Users\SystemX\Downloads\quasar-project\quasar-project\dist

Если сгенерировалcя aab то что-то было сделано не так


если мы хотим релизный апк то выполняем подписание при помощи ключа
