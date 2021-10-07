---
title: [CVE-2019-9674 Zip bomb vulnerability in CPython Lib/zipfile.py]
date: 2021-09-30 16:39:10
tags:
- vulnerability
- python
---

# Background

I reported a zip bomb vulnerability to the CPython community in 2019. Here are all the interesting resources and ideas.

[Issue Discussion on BPO](https://bugs.python.org/issue36462)

[CVE-2019-9674](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9674)

[Decompression pitfall I wrote for official documentation](https://docs.python.org/3/library/zipfile.html#decompression-pitfalls)

[Pull Request related it](https://github.com/python/cpython/pull/13378)

[PyCon Korea 2019 - Click Click Boom! Bombs Over Our Minds](https://www.youtube.com/watch?v=-S4JVQt6GX4&ab_channel=PyConKorea)

# zipfile analysis

According to Black Hat's  [Cara Marie](https://www.blackhat.com/docs/us-16/materials/us-16-Marie-I-Came-to-Drop-Bombs-Auditing-The-Compression-Algorithm-Weapons-Cache.pdf) research, there are some solutions against Zip Bomb. By limiting the size of the block to be read at a time, if there is still data remaining after the block that needs to be decompressed after reading this block, it is considered that it is possible to be a Zip Bomb.

Below is [Cara Marie](https://www.blackhat.com/docs/us-16/materials/us-16-Marie-I-Came-to-Drop-Bombs-Auditing-The-Compression-Algorithm-Weapons-Cache.pdf) code

```python
import zlib
def decompress(data, maxsize=1024000):
    dec = zlib.decompressobj()
    data = dec.decompress(data, maxsize)
    if dec.unconsumed_tail:
        raise ValueError("Possible bomb")
    del dec
    return data
```

As you can see, the strategy to defeating the zip bomb is by limiting a block, in this case, is max size 102400. However, we take a look at the Python standard library, [zipfile](https://github.com/python/cpython/blob/3.9/Lib/zipfile.py).


---

According to Cara Marie's approach, we try to figure out the difference between zipfile and zlib and **why we can't use zipfile directly for preventing zip bombs**, so we started to study zipfile source code.


# [zipfile](https://github.com/python/cpython/blob/master/Lib/zipfile.py)

Since I focus on the zip format and pick the most commonly used algorithm, DEFLATED algorithm. Inside the zipfile, we can see the location of unzipped function, starting at line `702`, getting the zlib object, and finally returning the object.

**[zlib.decompressobj(-15)](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L702-L716)**


```python
def _get_decompressor(compress_type):
    if compress_type == ZIP_STORED:
        return None
    elif compress_type == ZIP_DEFLATED:
        return zlib.decompressobj(-15)
    elif compress_type == ZIP_BZIP2:
        return bz2.BZ2Decompressor()
    elif compress_type == ZIP_LZMA:
        return LZMADecompressor()
    else:
        descr = compressor_names.get(compress_type)
        if descr:
            raise NotImplementedError("compression type %d (%s)" % (compress_type, descr))
        else:
            raise NotImplementedError("compression type %d" % (compress_type,))
```

From the above code, we can know that the zipfile is based on what zlib does. So we have to deep dive into what zlib did?

# [zlib](https://docs.python.org/3/library/zlib.html)

According to the zlib documentation

> There are two ways to compression and decompression, .compress() and .decompress() will fit all files into memory at once. In contrast to the method of the object. It using .compressobj() and .decompressobj() which won’t fit into memory at once.

There are two ways to compress/decompress.

1. .compress() and .decompress() will put the entire file into memory at a time
2. .compressobj() and .decompressobj() separate the file , compress/decompress one block at a time

---

However, the official documentation does not clearly explain how to use the API to decompress files. The purpose of this method is to obtain the file data stream and decompress it through the Low-Level method. And we went back to the zipfile module and found that they had already done the decompression of zlib, so we planned to apply the patch for zipfile first.

In the way that zipfile belongs to `decompressobj`, we have the first way to accumulate chunks. As long as we can find out where to do the decompression of chunks, we accumulate it and give a threshold. If it exceeds, then consider that it is possible to be the zip bomb.


# Get back at the zipfile

1. Starting with the object

[Line 706](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L706)
```python
return zlib.decompressobj(-15)
```

[Line 791](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L791)

It is the place where the class of zlib.decompressobj(-15) object is obtained and initialized.


which belongs to ZipExtFile class
```python
    def __init__(self, fileobj, mode, zipinfo, decrypter=None,close_fileobj=False):
```

Let's find out what `fileobj` is

[Line 1545](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L1545)

```python
return ZipExtFile(zef_file, mode, zinfo, zd, True)
```

Return the class, and use zef_file, then follow zef_file



[Line 719](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L719)

_SharedFile being initialized

```python
    def __init__(self, file, pos, close, lock, writing):
```

Here we know that when zlib is decompressed, you can't start decompressing directly to Streaming, and you need to skip the file encoding in front of the zip file.

[Line 759](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L759)

In class _Tellable: to initialize the position of the indicator that gets the file descriptor
```python
    def __init__(self, fp):
        self.fp = fp
        self.offset = 0
```

and then

[Line 977](https://github.com/python/cpython/blob/f2320b37d9c85d8ddfc0c6afa81b77cd5f6e5ef2/Lib/zipfile.py#L977-L984)


```python

        elif self._compress_type == ZIP_DEFLATED:
            n = max(n, self.MIN_READ_SIZE)
            data = self._decompressor.decompress(data, n)
            self._eof = (self._decompressor.eof or
                         self._compress_left <= 0 and
                         not self._decompressor.unconsumed_tail)
            if self._eof:
                data += self._decompressor.flush()

```

We observed that after choosing to use the ZIP_DEFLATED compression algorithm, we did a function max to get n.

# Key Point
```python
max(n, self.MIN_READ_SIZE)
```

When you use zlib.decompressobj as a block, how big is your block?
, self.MIN_READ_SIZE is preset to 4096 bytes, which is the size of a page in the operating system.

## Cara Marie's solution

```python
import zlib
def decompress(data, maxsize=1024000):
    dec = zlib.decompressobj()
    data = dec.decompress(data, maxsize)
    if dec.unconsumed_tail:
        raise ValueError("Possible bomb")
    del dec
    return data
```

## It sets maxsize to 102400 bytes

According to the official document

> Decompress.decompress(data, max_length=0)
> Decompress data, returning a bytes object containing the uncompressed data corresponding to at least part of the data in the string. This data should be concatenated to the output produced by any preceding calls to the decompress() method. Some of the input data may be preserved in internal buffers for later processing.
>
> If the optional parameter max_length is non-zero then the return value will be no longer than max_length. This may mean that not all of the compressed input can be processed, and unconsumed data will be stored in the attribute unconsumed_tail. This byte string must be passed to a subsequent call to decompress() if decompression is to continue. If max_length is zero then the whole input is decompressed, and unconsumed_tail is empty.
>
> Changed in version 3.6: max_length can be used as a keyword argument.


Max_length represents the file block size that can be read into the memory at a time and is marked with unconsumed_tail to see if any remaining files need to be decompressed.

Therefore, his idea is more than 102400 bytes. If there is any remaining data, it means there may be a zip bomb.

![](https://i.imgur.com/7schHy0.png)

