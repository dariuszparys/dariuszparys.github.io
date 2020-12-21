---
title: "Powershell EncodedCommand From Bash or Python"
date: 2020-12-15T06:57:39+01:00
draft: false
---

Powershell supports a feature to invoke commands encoded as base64 strings. This feature is handy if your command is assembled through another script in order to be executed and contains a lot of parameters where quoting and special character encoding can become tricky.

## Encoding commands

In order to use the EncodedCommand flag like stated in the [official documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pwsh?view=powershell-7.1#-encodedcommand---e---ec) you need to encode the string at *UTF-16LE*.

>**Update** after reporting the [issue with the documentation](https://github.com/MicrosoftDocs/PowerShell-Docs/issues/7059) it was quickly fixed. I shortened this blog post as it is now correctly documented.

## EncodedCommand with Python

```python
import base64

to_encode = 'docker build -t test:latest --build-arg TOKEN=''Test$$1234'' .'
encoded_bytes = base64.b64encode(to_encode.encode('utf-16-le'))
encoded_string = str(encoded_bytes, 'utf-8')
print(encoded_string)
```

## EncodedCommand with Bash

```bash
to_encoded_command() {
    local command_to_encode=$1
    echo "${command_to_encode}" | iconv --to-code UTF-16LE | base64 -w0
}

to_encoded_command 'docker build -t test:latest --build-arg TOKEN=''Test$$1234'' .'
```

## Sources

1. https://stackoverflow.com/questions/36550038/in-utf-16-utf-16be-utf-16le-is-the-endian-of-utf-16-the-computers-endianness
2. https://en.wikipedia.org/w/index.php?title=UTF-16&oldid=714413633#Byte_order_encoding_schemes
3. https://github.com/MicrosoftDocs/PowerShell-Docs/issues/7059