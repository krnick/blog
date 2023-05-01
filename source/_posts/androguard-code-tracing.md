---
title: androguard code tracing
date: 2022-04-22 15:12:30
tags:
---

# androguard code trace


## androguard Usage

This is the most common way how we use androguard.

```python=
from androguard.misc import AnalyzeAPK


apk, dalvikvmformat, analysis = AnalyzeAPK(apk_filepath)
```

---

## Entry Point

Let' us start from the ***AnalyzeAPK***.

## AnalyzeAPK(androguard/misc.py)

Responsible for the generation of the three objects below.

* apk
* dalvikvmformat
* analysis

---

### [apk object](https://androguard.readthedocs.io/en/latest/api/androguard.core.bytecodes.html#androguard.core.bytecodes.apk.APK)

You can get information like the package name, permissions from the AndroidManifest.xml, which generated by class APK().

**androguard/misc.py**

```python=
a = APK(_file, raw=raw)
```


**androguard/core/bytecodes/apk.py**
```python=
class APK():
```

It will parse files in APK through Python zipfile module. This APK object is responsible for decompressing "AndroidManifest.xml" in APK.

Implementation function:

**androguard/core/bytecodes/apk.py**

```python=
if not skip_analysis:
    self._apk_analysis()
```

```python=
def _apk_analysis(self)
```

ASRC class

    for decoding resources.arsc

AXML class

    for decoding AndroidManifest.xml and all other XML files
    
Inside this method, it will get the information below.

* AXML
* Permissions
* Each tag in xml

The following video records the process of making an apk object.

{%youtube CpwzaB6U7BU %}


---

### [dalvikvmformat Object](https://androguard.readthedocs.io/en/latest/api/androguard.core.bytecodes.html#androguard.core.bytecodes.dvm.DalvikVMFormat)

The DalvikVMFormat corresponds to the DEX file found inside the APK file. You can get classes, methods or strings from the DEX file.


**androguard/misc.py**

```python=
d = []
dx = Analysis()
for dex in a.get_all_dex():
    df = DalvikVMFormat(dex, using_api=a.get_target_sdk_version())
    dx.add(df)
    d.append(df)
    df.set_decompiler(decompiler.DecompilerDAD(d, dx))

dx.create_xref()

return a, d, dx
```



---

### [Analysis Object](https://androguard.readthedocs.io/en/latest/api/androguard.core.analysis.html#androguard.core.analysis.analysis.Analysis)

The Analysis object should be used instead, as it contains special classes, which link information about the classes.dex and can even handle many DEX files at once.

The Analysis contains a lot of information about (multiple) DalvikVMFormat objects Features are for example XREFs between Classes, Methods, Fields, and Strings. Yet another part is the creation of BasicBlocks, which is important in the usage of the Androguard Decompiler.


**quark/androguard/misc.py**

```python=
dx = Analysis()
```

**androguard/core/analysis/analysis.py**

```python=
class Analysis:
```

Find information inside DEX file:

* DalvikVMFormat objects
* classname
* string
* methods

[EXTERNAL method](https://androguard.readthedocs.io/en/latest/intro/gettingstarted.html)

It is called the native API.

[XREFs](https://androguard.readthedocs.io/en/latest/intro/gettingstarted.html#xrefs)

Classes, Methods, Fields, and Strings.

---

Initialize d and dx.

![](https://i.imgur.com/uBkdkWj.png)


Prepare for the following code below.

```python=
for dex in a.get_all_dex():
    df = DalvikVMFormat(dex, using_api=a.get_target_sdk_version())
    dx.add(df)
    d.append(df)
    df.set_decompiler(decompiler.DecompilerDAD(d, dx))

dx.create_xref()
```

Param dex will return the raw data of all classes dex files.

```python=
get_all_dex()

    dexre = re.compile(r"classes(\d*).dex")
    return filter(lambda x: dexre.match(x), self.get_files())
```

`get_all_dex` get all the file names in the zip, and get all the class.dex files through regular expressions, it may be classes.dex or classes[0-9]+.dex.


**quark/androguard/misc.py**

```python=
df = DalvikVMFormat(dex, using_api=a.get_target_sdk_version())
```

DalvikVMFormat takes the binary content of each DEX as a parameter, and initializes the class DalvikVMFormat according to the API version used.


**androguard/core/bytecodes/dvm.py**

```python=
class DalvikVMFormat(bytecode.BuffHandle):
    """
    This class can parse a classes.dex file of an Android application (APK).

    :param buff: a string which represents the classes.dex file
    :param decompiler: associate a decompiler object to display the java source code
    :type buff: bytes
    :type decompiler: object

    example::

        d = DalvikVMFormat( read("classes.dex") )
    """

    def __init__(self, buff, decompiler=None, config=None, using_api=None):
        # to allow to pass apk object ==> we do not need to pass additionally target version
        if isinstance(buff, APK):
            self.api_version = buff.get_target_sdk_version()
            buff = buff.get_dex()  # getting dex from APK file
        elif using_api:
            self.api_version = using_api
        else:
            self.api_version = CONF["DEFAULT_API"]

        super().__init__(buff)
        self._flush()

        self.CM = ClassManager(self)
        self.CM.set_decompiler(decompiler)

        self._preload(buff)
        self._load(buff)
```

This class can parse a classes.dex file of an Android application (APK).



**androguard/core/bytecodes/dvm.py**

* ClassManager

This class is used to access to all elements (strings, type, proto ...) of the dex format based on their offset or index.


* set_decompiler

Setting the disassembled tool used for disassembling an apk such as Jadx or DAD.


Last one:

**androguard/core/analysis/analysis.py**

```python=
dx.create_xref()
```

Create Class, Method, String, and Field crossreferences for all classes in the Analysis.

If you are using multiple DEX files, this function must be called when all DEX files are added. If you call the function after every DEX file, it will only work for the first time.

This function is already called in the object Analysis, so we don't need to call it again.


The following video records the process of making an DalvikVMFormat object.

{%youtube Q3oCMVyhGLw %}


# Important


Only when calling the `get_source_method` function in [DecompilerDAD](https://androguard.readthedocs.io/en/latest/api/androguard.decompiler.html#androguard.decompiler.decompiler.DecompilerDAD) is it considered that disassemble an apk.

In fact, we only use the information in the Dex file and the method of cross-references, as well as the information about android permission. We did not do the disassemble an apk.





