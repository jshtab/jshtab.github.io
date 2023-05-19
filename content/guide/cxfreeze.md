---
title: "Exploring cx_Freeze Applications"
date: 2023-05-19
description: 
tags:
 - cx_Freeze
 - reversing
 - python
 - software
 - disassembly
 - pyc
 - marshal
 - malware analysis
draft: true
---

# Exploring cx_Freeze Applications

While analyzing applications in the wild, you might come across a Python application bundled with [cx_Freeze]. It creates isolated, portable Python applications by including copies of everything the code needs. The final application is a ready-to-go .exe file that is attractive to typical users.

While cx_Freeze is used to package legitimate software, it and similar tools (like [PyInstaller]) are also used for disguising malware as legitimate executables. Knowing how to get at the actual Python code out of them means finding out more, and faster.

[cx_Freeze]: https://cx-freeze.readthedocs.io/en/latest/
[PyInstaller]: https://pyinstaller.org/en/stable/

## Understanding cx_Freeze

`cxfreeze` takes Python code and generates a directory containing the final app. The final app is bundled with a copy of whatever Python interpreter was available on the system and a copy of any libraries the code also needs.

Here's an example of what cx_Freeze generates on Windows:
```txt
dist
├── app_name_here.exe
├── python311.dll
├── python3.dll
├── vcruntime140.dll
└── lib
    ├── library.zip
    └── ...
```

The file used to launch the application is `app_name_here.exe`. It doesn't contain anything interesting, just enough to start Python and run the real program. It usually stays the same for all applications built with the same version of cx_Freeze. [^1]

[^1]: It can change depending on what settings were used when running `cxfreeze`. Don't rely on the checksum for detecting applications bundled with it.

The actual code is stored in the `lib` folder. Python packages required to run the program are bundled here, along with `library.zip`, which stores the program's code.

## Extracting compiled bytecode

The start of the Python program is in `library.zip`, usually in a file named `app_name_here__main__.pyc` containing **Python bytecode**. Getting the original source code from this file is usually impossible, so instead the bytecode is analyzed.

`.pyc` files are made of two parts: the header number and the code object.[^2] In modern versions of Python, the header is 16 bytes long. In Python 3.4 or lower, it was 8 bytes long. Use the correct number below.

[^2]: Technically this is only true for CPython, but because CPython is so common, assume that interpreter is used. It's *possible* but rare to bundle Cython/PyPy with cx_Freeze.

The rest of the file is a code object dumped to binary data using `marshal`. To extract it, open up a Python interpreter. Preferably inside of a sandbox or lab since `marshal` can be [exploited](https://github.com/python/cpython/issues/85380). Be careful, *especially* when doing this to real malware.

To start, import these modules:

```python
>>> import marshal
>>> import dis
```

To extract the data, open the bytecode file in binary reading mode using `open()`, skip past the header, and read the rest of the file using `marshal`.

```python
>>> with open('app_name_here__main__.pyc', 'rb') as file:
...     file.seek(16)  # header is 16 bytes in Python 3.11
...     bytecode = marshal.load(file) 
```

Finally, disassemble the code object by running `dis`:

```python
>>> dis.dis(bytecode)
  0           0 RESUME                   0

  1           2 PUSH_NULL
              4 LOAD_NAME                0 (print)
              6 LOAD_CONST               0 ('hello world!')
              8 PRECALL                  1
             12 CALL                     1
             22 POP_TOP
             24 LOAD_CONST               1 (None)
             26 RETURN_VALUE
```

## Understanding Python disassembly

If that example above looks unfamiliar, welcome to Python disassembly. Explaining everything here is excessive, so here's a video explaining the basics:

{{< youtube PJ16cdc0YKM >}}

Specific technical information, like opcode documentation for specific versions of Python exist in that version's [manual][PyManual].

[PyManual]: https://docs.python.org/3/library/dis.html

## Further reading

Here are some additional videos and websites for learning more about this bytecode disassembly and Python malware analysis.

- [In-depth bytecode talk at EuroPython 2016](https://www.youtube.com/watch?v=GNPKBICTF2w)
- [Analyzing obfuscated Python malware](https://www.youtube.com/watch?v=XZj87tKIlik)
