# ConsumerProGuardFilesTest

Example repository used to test whether consumer ProGuard files defined by transitive dependencies are used when ProGuarding a consuming application.

# Project Structure

The project contains an Android application module (app) and two Android library modules (library1 and library2). Both libraries contain a single class and a `proguard-rules.pro` ProGuard file that protects this class from minification and obfuscation. Both libraries expose this rules file via the `consumerProguardFiles` property. According to the [Android Gradle plugin documentation](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.ProductFlavor.html#com.android.build.gradle.internal.dsl.ProductFlavor:consumerProguardFiles), this property represents:

> ProGuard rule files to be included in the published AAR.
> 
> These proguard rule files will then be used by any application project that consumes the AAR (if ProGuard is enabled).

# Experiments

## Verifying behavior with two explicit dependencies

- Checkout the `app_depends_on_both_libraries_explicitly` branch.

On this branch, app dependencies are declared as follows:

```groovy
dependencies {
    compile project(':library1')
    compile project(':library2')
}
```

and neither library1 nor library2 declare any dependencies.

- Execute the command `./gradlew clean`
- Execute the command `./gradlew app:assembleDebug` (running these commands separately avoids some directory sync issues)
- Open the generated file located at `app/build/outputs/mapping/seeds.txt`

This file contains a list of classes and methods that were not obfuscated.

- Observe that the following classes were not obfuscated:
	- com.github.stkent.library1.TestClass1
	- com.github.stkent.library2.TestClass2

## Verifying behavior with one explicit and one implicit dependency

- Checkout the `app_depends_on_library1_explicitly_and_library2_implicitly` branch.

On this branch, app dependencies are declared as follows:

```groovy
dependencies {
    compile project(':library1')
}
```

library1 dependencies are declared as follows:

```groovy
dependencies {
    compile project(':library2')
}
```

and library2 declares no dependencies.

- Execute the command `./gradlew clean`
- Execute the command `./gradlew app:assembleDebug`
- Open the generated file located at `app/build/outputs/mapping/seeds.txt`
- Observe that the following classes were still not obfuscated:
	- com.github.stkent.library1.TestClass1
	- com.github.stkent.library2.TestClass2

# Conclusion

Based on the fact that `TestClass2` is protected from obfuscation when declared as an implicit dependency of the app module only, it would appear that ProGuard configuration set via the `consumerProguardFiles` is respected for transitive dependencies of Android applications. A further test can confirm that `consumerProguardFiles` is playing a key role here:

- Checkout the `app_depends_on_library1_explicitly_and_library2_implicitly` branch.

On this branch, app dependencies are declared as follows:

```groovy
dependencies {
    compile project(':library1')
}
```

library1 dependencies are declared as follows:

```groovy
dependencies {
    compile project(':library2')
}
```

and library2 declares no dependencies.

- Comment out the line `consumerProguardFiles 'proguard-rules.pro'` from the `library2` `build.gradle` file.
- Execute the command `./gradlew clean`
- Execute the command `./gradlew app:assembleDebug`
- Open the generated file located at `app/build/outputs/mapping/seeds.txt`
- Observe that the following class was not obfuscated:
	- com.github.stkent.library1.TestClass1
- Observe that the following class was obfuscated:
	- com.github.stkent.library2.TestClass2

# License

    Copyright 2016 Stuart Kent
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
       http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

