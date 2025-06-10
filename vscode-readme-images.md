---
layout: blog-post
title: The proper way to include an image to the README of an VS Code extension
---
https://github.com/cschot/cschot.github.io/blob/4a883971d05e4fda6e07b832c5ff89341c559e43/_config.yml#L1
So I've been writing the [Codespaces Usage Monitor](https://marketplace.visualstudio.com/items?itemName=cschot.codespaces-usage-monitor)
extension for VS Code in VS Code in the past couple of days.
I was finished writing the code an came to the part of writing the README...

I wanted to include an image in the README. How do you do I that? I don't have much experience with the Markdown language yet, so I looked it up.
It seemed simple: just copy the image and paste it in the README.
So that is what I did.

It created this line for me:
#https://github.com/cschot/codespaces-usage-monitor/blob/063a19c8be7ab2257c6882d0dad1ffd80c9660a8/README.md?plain=1#L11
> The filename is automatically placed between anglebrackets (<...>) because it contained spaces.\
> Sidenote: The dutch word for screenshot is schermafbeelding

I went to the preview (CTRL/CMD + SHIFT + V). And there the image was!
So I thought everything was fine and went on to publish the extension.
I ran `vsce publish`. It returned the url from the VS Code Marketplace for this extension.
I went there and checked the results:

![Schermafbeelding 2025-06-08 232230](https://github.com/user-attachments/assets/16e9ac12-eadb-40b1-a1c8-702ce0833d74)

What happened here!? 
I opened the Devtools of my Chrome Browser (press F12)

It showed me that somehow the source URL of the image was changed to:
`https://github.com/cschot/codespaces-usage-monitor/raw/HEAD/<Schermafbeelding 2025-06-08 003332.png>`

I did not expect it to load images directly from GitHub. I thought it would serve them from the VSIX package file as they are getting packed in there by VSCE.
So I went to GitHub and copied the source URL of the image there:
> https://raw.githubusercontent.com/cschot/codespaces-usage-monitor/refs/heads/main/Schermafbeelding%202025-06-08%20003332.png
I pasted it into the README file:
#https://github.com/cschot/codespaces-usage-monitor/commit/c7a99dadc3bafbbdc0fc4a6644a60d3fd1b51949#diff-b335630551682c19a781afebcf4d07bf978fb1f8ac04c6bf87428ed5106870f5R11
I then published a new version of the extension to the VS Code marketplace.
And... No 😞. Again the image was not shown.
I went to the Chrome devtools again and it showed me that, this time, the source of the image was replaced with:
> https://github.com/cschot/codespaces-usage-monitor/raw/HEAD/<https:/raw.githubusercontent.com/cschot/codespaces-usage-monitor/refs/heads/main/Schermafbeelding%202025-06-08%20003332.png>
Somehow it really wants to prepend the image's source with: > https://github.com/cschot/codespaces-usage-monitor/raw/HEAD/
So I figured out that I needed to get the link from the image source from my GitHub HEAD branch.
Wich was: > https://github.com/cschot/codespaces-usage-monitor/raw/HEAD/Schermafbeelding%202025-06-08%20003332.png
I then copied the filename to the README and removed the anglebrackets (<...>):
#https://github.com/cschot/codespaces-usage-monitor/commit/8a65d08e69ca76946cf1bf96dac526f0ae8f39dd#diff-b335630551682c19a781afebcf4d07bf978fb1f8ac04c6bf87428ed5106870f5R11
And after publishing a new version to the VS Code Marketplace... Yes! It showed the image right away!

So I thought everything was fine. But then I went to see how the README was displayed in <https://vscode.dev> and <https://github.dev>.
Bummer! No image there:
![image](https://github.com/user-attachments/assets/80c6d453-c379-4956-af5c-21c12a55e82c)
So I checked the source of the image in devtools. It showed the same source here as it did at VS Code Marketplace.
So that was not the problem. I went on to the Network panel of devtools and saw that the request was redirected and blocked at the same time:
![image](https://github.com/user-attachments/assets/ed797aa5-ed2e-4756-917f-d6a478b0c4b4)

![image](https://github.com/user-attachments/assets/6f453aae-a244-4955-8124-579fe8a165e4)

When I checked some other plugins, I saw that most of them had no working images in the README.
But I found one that did and checked the source of the image:
> https://raw.githubusercontent.com/zengjianjay/vscode-starlark/master/images/highlight.png
Somehow images from the domain `raw.github.com` are loaded. But those from `github.com` are not.
I then changed the source of my image in devtools with the `raw.github.com` one, and it loaded right away.

To see why the image sources are rewritten by VSCE, you can check out it's source code here:
#https://github.com/microsoft/vscode-vsce/blob/main/src/main.ts
