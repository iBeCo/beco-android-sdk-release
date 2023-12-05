# beco-android-sdk-release
This repository will be used to host android sdk dependency on github

### Installing
* Add following lines to your gradle file:

```java

repositories {
    maven {url "https://raw.githubusercontent.com/iBeCo/beco-android-sdk-release/release"}
}


dependencies {
    compile 'com.beco:sdk:1.19'
}
```
