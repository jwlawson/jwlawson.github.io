---
layout: post
category: interest
title: Blender plugin to rescale images
---

After a trip to the US, I set about creating a video of the clips and photos from my time there. Having no experience of any video editing software I looked around for something free and open source, with Blender frequently coming up as a suggestion. Blender is best known for animating 3D models but also contains a fully functional video sequence editor (VSE).

After a little while playing aroud with it, the one thing that really annoyed me about blender's VSE was how it imported images. Unless the image you import is exactly the same size as the render target Blender will squash it and mangle it to that size. In the video I was looking to import a large portrait photo and pan from the bottom to the top, so I needed the image in its original aspect ratio and ideally at its full resolution. Initially I manually added transfrom layers to rescale the image back to its proper aspect ration, but quickly got bored of doing this and wrote an add-on to do it for me.

The finished video:
<iframe width="560" height="315" src="https://www.youtube.com/embed/df1EAIxAFVc" frameborder="0" allowfullscreen></iframe>
<!--end-excerpt-->

The add-on requires very few dependencies as I had no idea what might be available to a default install of blender. In fact `struct`, `imghdr` and `re` are only used by the `get_image_size` function, which could easily be replaced by `PIL` or similar if available.

```python
"""
Blender add-on which provides a function in the VSE to rescale images to their
original aspect ratio.

Due to the inclusion of code licensed under GPLv2, this too is licensed under
GPLv2. See: https://www.gnu.org/licenses/old-licenses/gpl-2.0.html
"""
import bpy
import os
import struct, imghdr, re
```

Any Blender add-on should follow the requirements set out [here][script-reqs]. As such we specify a `bl_info` dictionary containing a little information about the add-on. This script does very little and is only meant as a personal tool, so the information is kept to a minimum.

```python
bl_info = {
    "name": "Rescale VSE Images",
    "description": "Adds transform layer to image to rescale to original aspect",
    "author": "J Lawson",
    "version": (1, 0),
    "blender": (2, 77, 0),
    "location": "?",
    "warning": "", # used for warning icon and text in addons panel
    "wiki_url": "",
    "tracker_url": "",
    "support": "COMMUNITY",
    "category": "Sequencer"
    }
```

In order to rescale a provided image we need to get the dimensions of the image. I had no idea what python libraries were available to a standard blender install and so had a hunt around for some way of doing this without depending on too much.

Stack exchange is full of all sorts of code snippets, supposedly available under the MIT license with 'reasonable attribution'. However there is no guarantee that whoever posts on SE is actually the original author of the code, so they don't actually have the right to provide the code under that license.

There is a now unmaintained python library/program [Draco] which was licensed under GPLv2 and looks to have been abandoned sometime around 2004/05. In Draco's `image.py` there is a function which finds the size of an image by looking at the file header without needing any external dependencies. This function has now been posted on stack exchange and since improved and modified and now is included in a number of different projects. It is not at all clear how to interpret the license of this code now. The initial code was licensed with GPLv2, so all derivatives should also be GPLv2. However, once posted to SE many developers have assumed that the code is available under the much more unrestrictive MIT license.

```python
def get_image_size(fname):
    """
    Return (width, height) for a given img file content.

    This function is heavily based on a similar funtion in `image.py` from the
    Draco dynamic web content system in python. Draco is available under the
    GPLv2.

    There are a number of similar functions floating around stack exchange, with
    slight changes to functionality. The version included here is based on
    Yantao Xie's version found here:
    https://stackoverflow.com/questions/15800704/
        python-get-image-size-without-loading-image-into-memory
    """
    with open(fname, 'rb') as fhandle:
        head = fhandle.read(32)
        if len(head) != 32:
            return
        if imghdr.what(fname) == 'png':
            check = struct.unpack('>i', head[4:8])[0]
            if check != 0x0d0a1a0a:
                return
            width, height = struct.unpack('>ii', head[16:24])
        elif imghdr.what(fname) == 'gif':
            width, height = struct.unpack('<HH', head[6:10])
        elif imghdr.what(fname) == 'jpeg':
            try:
                fhandle.seek(0) # Read 0xff next
                size = 2
                ftype = 0
                while not 0xc0 <= ftype <= 0xcf:
                    fhandle.seek(size, 1)
                    byte = fhandle.read(1)
                    while ord(byte) == 0xff:
                        byte = fhandle.read(1)
                    ftype = ord(byte)
                    size = struct.unpack('>H', fhandle.read(2))[0] - 2
                # We are at a SOFn block
                fhandle.seek(1, 1)  # Skip `precision' byte.
                height, width = struct.unpack('>HH', fhandle.read(4))
            except Exception: #IGNORE:W0703
                return
        elif imghdr.what(fname) == 'pgm':
            header, width, height, maxval = re.search(
                b"(^P5\s(?:\s*#.*[\r\n])*"
                b"(\d+)\s(?:\s*#.*[\r\n])*"
                b"(\d+)\s(?:\s*#.*[\r\n])*"
                b"(\d+)\s(?:\s*#.*[\r\n]\s)*)", head).groups()
            width = int(width)
            height = int(height)
        elif imghdr.what(fname) == 'bmp':
            _, width, height, depth = re.search(
                b"((\d+)\sx\s"
                b"(\d+)\sx\s"
                b"(\d+))", str).groups()
            width = int(width)
            height = int(height)
        else:
            return
        return width, height
```

<div class="panel panel-info panel-body">
This code is roughly equivalent to
<div class="language-python highlighter-rouge"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">PIL</span> <span class="kn">import</span> <span class="n">Image</span>

<span class="k">def</span> <span class="nf">get_image_size</span><span class="p">(</span><span class="n">fname</span><span class="p">):</span>
    <span class="k">with</span> <span class="n">Image</span><span class="o">.</span><span class="nb">open</span><span class="p">(</span><span class="n">filename</span><span class="p">)</span> <span class="k">as</span> <span class="n">img</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">img</span><span class="o">.</span><span class="n">size</span>
</code></pre>
</div>
but does not require any dependencies. If you have <code class="highligher-rouge">PIL</code> installed, then it is almost certainly better to use this smaller, easier to read, easier to maintain version.
</div>

Now we finally get on with the add-on class. Each class needs a `poll` method, which is used to check whether the context is right to show the function, and an `execute` method which actually does whatever your function should do.

```python
class Rescale_VSE_Image(bpy.types.Operator):
    bl_label = 'Rescale VSE Image'
    bl_idname = 'sequencerextra.rescaleimage'
    bl_description = 'Adds transform layer to image to rescale to original aspect'
    bl_options = {'REGISTER', 'UNDO'}
```

This function should only be available in the video editor, and only when the selected strip is an image. There is no sense trying to resize videos or audio clips.

```python
    @classmethod
    def poll(cls, context):
    """ Ensure that the function is only available for images """
        scn = context.scene
        strip = scn.sequence_editor.active_strip
        if scn and scn.sequence_editor and strip:
            return strip.type in ('IMAGE')
        else:
            return False
```

We finally get to the method which resizes the image in the video editor. First we find the path to the currently selected image and use the `get_image_size` method above to find the original size for the image.

After being imported into blender the image is squished into the exact render size, so `render_x by render_y` with aspect ratio `asp_n = render_x / render_y`. The aim is to scale the image so that the aspect ratio is infact `asp_o = orig_x / orig_y`, so we scale the `x` coord by `asp_o / asp_n`, which in fact is the same as scaling by `render_y / render_x * orig_x / orig_y`.

The scaling is done by adding an effect strip. The call to `sequencer.effect_strip_add` automatically selects the newly added effect strip, so we use `active_strip` to get a reference to this transform in order to set the scale in it.

```python
    def execute(self, context):
        """
        1. Get the image path
        2. Find the size of the image
        3. Apply a transform to scale the image correctly
        """
        scn = context.scene
        strip = scn.sequence_editor.active_strip
        file = os.path.realpath(bpy.data.filepath + "/.." + strip.directory + strip.name)
        size = get_image_size(file)
        render_setting = scn.render
        render_x = render_setting.resolution_x
        render_y = render_setting.resolution_y
        bpy.ops.sequencer.effect_strip_add(type='TRANSFORM')
        transform = scn.sequence_editor.active_strip
        transform.scale_start_x = render_y/render_x*size[0]/size[1]
        transform.scale_start_y = 1.0
        return {'FINISHED'}
```

The following are blender specific functions which are called when installing the add-on. They register this class with blender, so you can actually use the function in the program.

```python
def register():
    bpy.utils.register_class(Rescale_VSE_Image)

def unregister():
    bpy.utils.unregister_class(Rescale_VSE_Image)
```

Full code is available as a [gist].

[gist]: https://gist.github.com/jwlawson/778c33fd393317dc235ff89292283ccc
[script-reqs]: https://wiki.blender.org/index.php/Dev:Py/Scripts/Guidelines/Addons
[Draco]: https://wiki.python.org/moin/Draco