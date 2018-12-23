---
title: "The **.animated** object format"
linktitle: "The ANIMATED object"
weight: 3
---

## ■ 1. Overview

The ANIMATED object format is a container format allowing you to reference other objects (B3D/CSV/X) and to apply animation to them. It also allows to just group other objects (including other ANIMATED objects) without animating them.

Animated objects can be used in CSV/RW routes (unless explicitly disallowed by some commands), as train exterior objects via the *extensions.cfg*, and as 3D cabs via the *panel.animated* file.

##### ● Basics

Animation is performed via the following primitives:

● State changes - basically allowing to switch between different objects at any time  
● Translation - moving objects in three independent directions  
● Rotation - rotating objects around three independent axes  
● Texture shifts - allowing to shift the texture coordinates of objects in two independent directions

##### ● A little formality

The file is a plain text file encoded in any arbitrary [encoding](/information/encodings.html), however, UTF-8 with a byte order mark is the preferred choice. The [parsing model](/information/numberformats.html) for numbers is **Strict**. The file name is arbitrary, but must have the extension **.animated**. The file is interpreted on a per-line basis, from top to bottom.

## ■ 2. Sections

##### ● The [Include] section

You can use the [Include] section to just include other objects, but without animating them. This allows you to use the ANIMATED object file as a container to group other objects. There can be any number of [Include] sections within the file.

{{% command %}}  
[Include]  
{{% /command %}}  
This starts the section.

{{% command %}}  
*FileName<sub>0</sub>*  
*FileName<sub>1</sub>*  
*FileName<sub>2</sub>*  
...  
{{% /command %}}  
Defines a series of B3D/CSV/X/ANIMATED objects that should be included as-is.

{{% command %}}  
**Position = X, Y, Z**  
{{% /command %}}  
This defines the position of the objects, basically allowing you to offset them with respect to the rest of the ANIMATED object file.

------

