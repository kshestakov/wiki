##  Конфликт версий Google Play Services в Gradle зависимостях. react-native-maps падает при подключении react-native-device-info / react-native-fcm и любого другого пакета, который имеет в своих зависимостях com.google.android.gms или com.google.firebase.

- Идем в папку node_modules и ищем там в модулях файлы build.gradle. В этих build.gradle ищем зависимости, начинающиеся с com.google.android.gms или com.google.firebase, запоминаем модули с этими зависимостями. (Допустим, это модули react-native-fcm, react-native-maps и react-native-device-info). 

_Пример:_

модуль __react-native-maps__, путь: node_modules/react-native-maps/lib/android/build.gradle
```java
dependencies {
  provided "com.facebook.react:react-native:+"
  compile "com.google.android.gms:play-services-base:10.2.0"
  compile "com.google.android.gms:play-services-maps:10.2.0"
}
```
модуль __react-native-fcm__, путь: node_modules/react-native-fcm/android/build.gradle
```java
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.facebook.react:react-native:+'
    compile 'com.google.firebase:firebase-core:+'
    compile 'com.google.firebase:firebase-messaging:+'
    compile 'me.leolin:ShortcutBadger:1.1.10@aar'
}
```
модуль __react-native-device-info__, путь: node_modules/react-native-device-info/android/build.gradle
```java
dependencies {
    compile 'com.facebook.react:react-native:+'
    compile 'com.google.android.gms:play-services-gcm:+'
}
```
- Идем в build.gradle своего приложения (android/app/build.gradle) и ищем блок с зависимостями в низу файла.

_Пример:_
```java
dependencies {
    compile project(':react-native-maps')
    compile project(':react-native-fcm')
    compile project(':react-native-device-info')
    compile project(':react-native-fetch-blob')
    compile project(':react-native-vkontakte-login')
    compile project(':react-native-image-picker')
    compile project(':react-native-fbsdk')
    compile project(':react-native-linear-gradient')
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
}
```
- В этом блоке надо найти модули, которые мы запомнили на первом шаге.
- Исключаем у этих модулей зависимости, начинающиеся с com.google.android.gms или com.google.firebase, следующим образом:
```java
dependencies {
    compile(project(':react-native-maps')){
        exclude group: 'com.google.android.gms'
    }
    compile (project(':react-native-fcm')){
        exclude group: "com.google.firebase" 
    }
    compile (project(':react-native-device-info')){
        exclude group: "com.google.android.gms" 
    }
    compile project(':react-native-fetch-blob')
    compile project(':react-native-vkontakte-login')
    compile project(':react-native-image-picker')
    compile project(':react-native-fbsdk')
    compile project(':react-native-linear-gradient')
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
}
```
- Принудительно (с флагом force = true) указываем зависимости (те самые, которые мы исключили на 4 шаге) с нужной нам версией:
```java
dependencies {
    compile(project(':react-native-maps')){
        exclude group: 'com.google.android.gms'
    }
    compile (project(':react-native-fcm')){
        exclude group: "com.google.firebase" 
    }
    compile (project(':react-native-device-info')){
        exclude group: "com.google.android.gms" 
    }
    compile project(':react-native-fetch-blob')
    compile project(':react-native-vkontakte-login')
    compile project(':react-native-image-picker')
    compile project(':react-native-fbsdk')
    compile project(':react-native-linear-gradient')
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
    compile ("com.google.android.gms:play-services-base:10.0.1") {
        force = true;
    }
    compile ("com.google.android.gms:play-services-maps:10.0.1") {
        force = true;
    }
    compile ("com.google.android.gms:play-services-gcm:10.0.1") {
        force = true;
    }
    compile ('com.google.firebase:firebase-core:10.0.1') {
        force = true;
    }
    compile ('com.google.firebase:firebase-messaging:10.0.1') {
        force = true;
    }
}
```
- Либо запускаем Build -> Clean Project в Android Studio, либо в терминале, находясь в папке android/ вашего проекта, пишем: ./gradlew clean
- Теперь проект должен сбилдиться и завестись без ошибок.

## Конфликт зависимостей. Авторазрешение.

- Ссылка: https://github.com/facebook/react-native/issues/14223#issuecomment-304509104

## Падает релизная сборка приложения (дебаговая работает нормально) без объяснения причин после разрешения зависимостей с Google Play Services

1. Попробовать поменять версии в build.gradle пакета react-native-facebook-sdk с 23 на 25 в полях:
- compileSdkVersion
- targetSdkVersion

2. Поиграться с зависимостями в своем build.gradle (android/app/build.gradle), попробовать следующий вариант:
- compileSdkVersion 25
- buildToolsVersion '25.0.2'
- compile "com.android.support:appcompat-v7:25.0.1"

## Firebase не хочет заводиться на iOS, если устанавливать через Cocoapods

1. Выполнить pod update после pod install