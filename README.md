# Quasar-build-to-apk


### <a id="title1"> Команды для подписания апк (ключ)</a>
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




## Windows

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
cordova build android --release -- --packageType=apk
```

И апк должен был сгенерироваться по указанному пути

Если не сгенерировалcя - что-то было сделано не так

если мы хотим подписать апк то выполняем подписание при помощи [ключа](#title1)




## Ubuntu

В консоли Android Studio

```bash
 sudo npm install -g cordova
```

```bash
 sudo apt install gradle
```

Ставим openjdk, если нужно - еще и обычную Java

```bash
sudo snap install openjdk
```

```bash
 cd src-cordova
```

```bash
 cordova platform remove android
cordova platform add android
```

```bash
cordova build android --release -- --packageType=apk
```
На этом этапе нужно будет настроить переменные которые указаны после выполнения команды (если ошибка)
можно настраивать переменные используя nano (recomended), либо напрямую вписав в консоль используя export 

```bash
nano ~/.bashrc
```

переменные которые я настроил, для примера: 

```nano
# Настройка переменных окружения для Java JDK
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64


export PATH=$PATH:$JAVA_HOME/bin

# Настройка переменных окружения для Android SDK
export ANDROID_HOME=/home/noort/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME

export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools


# Настройка переменной окружения для Gradle
export GRADLE_HOME=/opt/gradle-7.0
export PATH=$PATH:$GRADLE_HOME/bin
```

после сохранения файла вписываем в консоль для обновления:

```bash
source ~/.bashrc
```
И теперь можно билдить снова. Здесь нужно помнить что есть разные флаги как info или debug которые будут выводить доп. информацию (при необходимости можно использовать)

Если возникают ошибки с Gradle или openjdk - можно попробовать установить другие версии из других источников
Но большая часть ошибок возникает из-за неправильной настройки переменных.

```bash
cordova build android --release -- --packageType=apk
```

если мы хотим подписать апк то выполняем подписание при помощи [ключа](#title1).





## Полезные ссылки (in progress):

https://quasar.dev/quasar-cli-vite/developing-cordova-apps/build-commands/

https://help.ubuntu.ru/wiki/nano



