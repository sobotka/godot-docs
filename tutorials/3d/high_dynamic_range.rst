.. _doc_high_dynamic_range:

Light Transport in Game Engines
===============================

Introduction
------------

Normally, an artist does all the 3D modelling, then all the texturing,
looks at their awesome looking model in the 3D DCC and says "looks
fantastic, ready for integration!" then goes into the game, lighting is
setup and the game runs.

So at what point does all this "HDR" business come into play? To understand
the answer, we need to look at how displays behave.

Your display outputs linear light ratios from some maximum to some minimum
intensity. Modern game engines perform complex math linear light values on
their respective scenes. So what's the problem?

The display has a limited range of intensity, depending on the display type.
The game engine renders to an unlimited range of intensity values, however.
While "maximum intensity" means something to an sRGB display, it has no bearing
in the game engine's math world; there is only a massive range of intensity values
generated per frame of rendering.

This means that some transformation of the scene light intensity, also known
as _scene referred_ light ratios, need to be transformed and mapped to fit
within the particular output range of the chosen display. This can be most
easily understood if we consider virtually photographing our game engine scene
through a virtual camera. Here, our virtual camera would apply a particular
camera rendering transform to the scene data, and the output would be ready
for display on a particular display type.

Computer Displays
-----------------

Almost all displays require a nonlinear encoding for the code values sent
to them. The display in turn, using it's unique transfer characteristic,
"decodes" the code value into linear light ratios of output, and projects
the ratios out of the uniquely coloured lights at each reddish, greenish,
and blueish emission site.

For a majority of computer displays, the specifications of the display are
outlined in accordance with IEC 61966-2-1, also known as the 1996 sRGB specification.
This specification outlines how an sRGB display is to behave, including the
color of the lights in the LED pixels as well as the transfer characteristics
of the input (OETF) and output (EOTF).

Not all displays use the same OETF and EOTF as a computer display however,
for example, television broadcast displays use the BT.1886 EOTF. It should
be noted that Godot is limited in its pixel management chain,
and only supports sRGB displays currently.

The sRGB standard is based around the nonlinear relationship between the current
to light output of common desktop computing CRT displays.

.. image:: img/hdr_gamma.png

The mathematics of a scene referred model require that we multiply the scene by different
values to adjust the intensities and exposure to different light ranges.
The transfer function of the display cannot appropriately render
the wider dynamic range of the game engine's scene output using the simple
transfer function of the display however, and therefore a more complex approach
to encoding is required.

Scene Linear & Asset Pipelines
------------------------------

Working in scene linear sRGB is not as simple as just pressing a switch. First,
imported image assets must be converted to linear light ratios on import. Even
when linearized, those assets may not be perfectly well suited for use as textures,
depending on how they were generated.

There are two ways to do this:

sRGB Transfer Function to Display Linear Ratios on Image Import
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the easiest, but not the most ideal, method of using sRGB assets.
One issue with this is loss of quality. Using 8 bit per channel to represent
linear light ratios is not a sufficient but depth to quantise the values correctly.
These textures might later be compressed too, which can exacerbate the problem.

Hardware sRGB Transfer Function to Display Linear Conversion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The GPU will do the conversion after reading the
texel using floating point. This works fine on PC and consoles, but most
mobile devices do no support it, or do not support it on compressed
texture format (iOS for example).

Scene Linear to Display Referred Nonlinear
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After all the rendering is done, the scene linear render requires transforming
to a suitable output such as an sRGB display. To do this, enable sRGB conversion in the
current :ref:`Environment <class_Environment>` (more on that below).

Keep in mind that sRGB -> Display Linear and Display Linear -> sRGB conversions
must always be **both** enabled. Failing to enable one of them will
result in horrible visuals suitable only for avant-garde experimental
indie games.

Parameters of HDR
-----------------

HDR setting can be found in the :ref:`Environment <class_Environment>`
resource. These are found most of the time inside a
:ref:`WorldEnvironment <class_WorldEnvironment>`
node or set in a camera. For more information see
:ref:`doc_environment_and_post_processing`.
