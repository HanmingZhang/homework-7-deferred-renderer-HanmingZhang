# Deferred Rendering & Post Processing


## [Demo Link](https://hanmingzhang.github.io/homework-7-deferred-renderer-HanmingZhang/)


## Overview
This project is an exercise about deferred rendering and post processing based on UPenn's CIS-566 Procedural Graphics using WebGL 2.0. 

One 32-bit RGBA texture and one 8-bit RGBA texutre are used as G-buffers, here is the structure of G-buffer:

![](./img/gbuffer.jpg)

The result we get after the first render to g-buffer pass:

![](./img/albedo_from_texture.jpg) | ![](./img/depth_camera_space.jpg) | ![](./img/normal.jpg)
----------------------|----------------------|----------------------
albedo | camera space depth | normal

HDR & LDR tone mapping is implemented to better preserve and recover color. In terms of some implementation details, I refer to [Filmic Worlds blog](http://filmicworlds.com/blog/filmic-tonemapping-operators/).

Here are three comparsion of different tone mapping methods:

![](./img/hdr_optimizedFormulaByJHandRBD.jpg) | ![](./img/hdr_Reinhard.jpg) | ![](./img/hdr_uncharted2.jpg)
----------------------|----------------------|----------------------
optimized formula by Jim Hejl and Richard Burgess-Dawson | Reinhard | Uncharted 2 operator


## Post-Processing

Here is the original render, just for the comparison after post processing.

![](./img/before_post_process.jpg)


### **Bloom**

The first pass of Bloom post processing is the brightness filter, which is later used as an input of Gaussian Blur pass(former by horizontal gaussian blur pass and then vertical gaussian blur pass). After that, I just combine it with the original render and get the final render. I made this base on my pervious [CIS-565 WebGL Clustered Deferred and Forward-Plus rendering project](https://github.com/HanmingZhang/Project5-WebGL-Clustered-Deferred-Forward-Plus).


![](./img/bloom_BrightnessFilter.jpg) | ![](./img/bloom_HorizontalGaussianBlur.jpg) | ![](./img/bloom_VerticalAndHorizontalGaussianBlur.jpg)
----------------------|----------------------|----------------------
brightness filter pass | horizontal blur pass | vertical and horizontal blur pass

and finally, after we combine it with the original render, we have

![](./img/bloom_final.jpg)


### **God Ray**
This effect requires an "occlusion pre-pass" as the input framebuffer. This means rendering all geometry as black ( (0, 0, 0), representing occlusion of light source ) and light source as normally would, which should yield a framebuffer looking something like [this](http://fabiensanglard.net/lightScattering/tutorial1LightAndOccluder.JPG). The godRay shader (different from the occlusion pre-pass shader) then computes a screen-space direction from a given fragment towards the light source(s). Repeatedly sample your image n times, stepping some amount along the light direction, accumulating the sampled color to effectively perform a blur. That should yield something like [this](http://fabiensanglard.net/lightScattering/tutorial2LightScattering.JPG). From there, blend these blurred rays with the framebuffer containing the actual rendered scene and get the final render.

Here are the results after each pass,

![](./img/godray_occlusion.jpg) | ![](./img/godray_after_first_pass.jpg) 
----------------------|----------------------
occlusion pass | accumulating pass 

and the final render,

![](./img/godray_final.jpg)


### **Cartoon**
This post processing effect is base on my previous [CIS565 BabylonJS Project](https://github.com/HanmingZhang/Babylon.js). Basically, there are two parallel post processing pipelines. The first one use Sobel filter to detection edges and draw them as black lines. The second one use a Kuwahara post processing effect to get some "Cartoon" feeling and then sample a paper texutre, blending it with the previous Kuwahara effect. Finally, we combine these two frames in a multiply way and add a frame to make it more "Cartoon"-like.

Here are the results after each pass,

![](./img/cartoon_edge.jpg) | ![](./img/cartoon_kuwahara.jpg) | ![](./img/cartoon_kuwahara_with_paper.jpg) 
----------------------|----------------------|----------------------
Edge detect(sobel filter) | Kuwahara | Paper effect

and the final render,

![](./img/cartoon_final.jpg)


### **Digital Rain**
This effect is inspired by the [BabylonJS Digital Rain Effect](https://doc.babylonjs.com/extensions/digitalrainpostprocess). Bascially, we divide the screen accroding one single font width and make them tiles, then fill these tiles with "fonts" sampling from the texture.

Here is the final render,

![](./img/digitalRain.jpg)
