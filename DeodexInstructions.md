# Introduction #

As of v1.4.0, deodexing has been greatly simplified. You should no longer need to specify additional classpath entries via the -c option.

Deodexing is now as simple as

```
baksmali -a <api_level> -x <odex_file> -d <framework_dir>
```

If you find that you have to use the -c option for something, please let me know so I can look into it (and hopefully fix it).


# General information about odex files #

In short, an odex file is an optimized version of a classes.dex file that has optimizations that are device specific. In particular, an odex file has dependencies on every "BOOTCLASSPATH" file that is loaded when it is generated. The odex file is only valid when used with these exact BOOTCLASSPATH files. dalvik enforces this by storing a checksum of each file that the odex file is dependent on, and ensuring that the checksum for each file matches when the odex file is loaded.

The BOOTCLASSPATH is simply a list of the jars/apk from which classes can be loaded, in addition to the main apk/jar that is loaded. A normal android system has 5 jars in it's base BOOTCLASSPATH - core.jar, ext.jar, framework.jar, android.policy.jar and services.jar. These can all be found in /system/framework. However, some apks have dependencies on additional jar or apks files beyond that of the base 5 jars. For example, for applications that use google maps, com.google.android.maps.jar will be appended to the BOOTCLASSPATH for that application's apk.

These odex dependencies make life a bit difficult for a couple of reasons. For one - you can't take an apk+odex file from one system image and run it on another system image (unless the other system image uses the exact same framework files). Another problem is that if you make any changes to any of the BOOTCLASSPATH files, it will invalidate every odex that depends on that file - basically every apk/jar on the device.

# Deodexing #

_The examples below assume you are using the baksmali wrapper script available on the downloads page to call baksmali. If you are calling the jar directly, you can replace "baksmali" with "java -jar baksmali.jar" instead_

The two primary options for deodexing are -x, and -d. -x is the flag that tells baksmali that you want to deodex something, and -d tells baksmali where to load dependencies from.

The dependencies can be in the form of an odex file, or a jar/apk with an embedded classes.dex file. Either works just as well.

For example, if you want to deodex Calculator.odex from an ICS image, and you have all the framework jars/odexes in a directory named framework:

```
baksmali -a 15 -x Calculator.odex -d framework -o Calculator
```

This will disassemble and deodex Calculator.odex, and place the resulting deodexed smali files into a directory named Calculator.

# Troubleshooting #

**Heap Space**

One issue you may run into is if it runs out of heap memory. The error might look something like "java.lang.OutOfMemoryError: Java heap space". To resolve this, you can
use the -Xmx parameter to increase the heap size. A reasonable size to try would be 512m. You can increase it further if needed, of course. If you are using the wrapper script to call baksmali, you can use -JXmx instead. For example

```
baksmali -JXmx512m -x blah.odex
```

Or if you are calling the jar directly with something like "java -jar baksmali.jar":

```
java -Xmx512m -jar baksmali.jar -x blah.odex
```