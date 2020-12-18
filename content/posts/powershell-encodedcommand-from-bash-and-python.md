---
title: "Powershell EncodedCommand From Bash or Python"
date: 2020-12-15T06:57:39+01:00
draft: false
---

Powershell supports a feature to invoke commands encoded as base64 strings. This feature is handy if your command is assembled through another script in order to be executed and contains a lot of parameters where quoting and special character encoding can become tricky.

## Encoding commands

In order to use the EncodedCommand flag like stated in the [official documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pwsh?view=powershell-7.1#-encodedcommand---e---ec) you need to encode the string at *UTF-16*.

## UTF 16 Little-endian

Windows is using per default little-endian order so if the documentation posts it is expecting *UTF-16* it actually expects *UTF-16-LE*.

Assume we have the following command we want to execute in powershell

```powershell
docker build -t test:latest --build-arg TOKEN='Test$$1234'
```

Following the sample of the documentation the conversion would look like this

```powershell
$command = 'docker build -t test:latest --build-arg TOKEN=''Test$$1234'' .'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)
Write-Host $encodedCommand
```

The result is a *UTF-16* encoded string.

```bash
ZABvAGMAawBlAHIAIABiAHUAaQBsAGQAIAAtAHQAIAB0AGUAcwB0ADoAbABhAHQAZQBzAHQAIAAtAC0AYgB1AGkAbABkAC0AYQByAGcAIABUAE8ASwBFAE4APQAnAFQAZQBzAHQAJAAkADEAMgAzADQAJwAgAC4A
```

It is *UTF-16* but more concrete it is *UTF-16-LE* but it isn't stated anywhere

If you try now to encode the string with the same encoding with Python you'll end up with something like this

```python
import base64

to_encode = 'docker build -t test:latest --build-arg TOKEN=''Test$$1234'' .'
encoded_bytes = base64.b64encode(to_encode.encode('utf-16'))
encoded_string = str(encoded_bytes, 'utf-8')
print(encoded_string)
```

and a result that differs from the powershell script above

```powershell
//5kAG8AYwBrAGUAcgAgAGIAdQBpAGwAZAAgAC0AdAAgAHQAZQBzAHQAOgBsAGEAdABlAHMAdAAgAC0ALQBiAHUAaQBsAGQALQBhAHIAZwAgAFQATwBLAEUATgA9AFQAZQBzAHQAJAAkADEAMgAzADQAIAAuAA==
```

This is a *UTF-16* string with an included byte order mark which let's the system choose the right interpretation in case it runs on an big endian platform. But that is not what *Powershell's EncodedCommand* expects. You'll have to use explicitly little-endian encoding to make that work.

## Correct Encoding with Python

```python
import base64

to_encode = 'docker build -t test:latest --build-arg TOKEN=''Test$$1234'' .'
encoded_bytes = base64.b64encode(to_encode.encode('utf-16-le'))
encoded_string = str(encoded_bytes, 'utf-8')
print(encoded_string)
```

## Correct Encoding with Bash

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