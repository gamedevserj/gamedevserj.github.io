---
layout: post
title:  "Invisibility effect shader tutorial"
date: 2020-10-04
tags: tutorial shader
---

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/Invisibility_final.gif">

This shader is a dissolve shader with some modifications – we want the part that is transparent in the dissolve shader to have the color of the background with some distortion applied to it.

Let's start by creating a new shader that we can do some tests on. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

void fragment()
{
	COLOR = texture(TEXTURE, UV * tiling + offset);
}
</pre></td></tr></tbody></table></code></div></div>

If we increase tiling we're going to see the image being squished. If your image import settings have "repeat" enabled you'll see multiple copies of the image. And if we change offset we'll see the image moving. 

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/tiling2_offset0.2.png">

Now we can see that UV coordinates determine how an image is applied onto the surface, and by manipulating those we can change the way it looks thus giving us a way to make it distorted. Just like with dissolve shader we're going to use a grayscale texture, but this time it will serve as an indicator to which parts of the image should be distorted more.  

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

<ins>uniform sampler2D noiseTexture;</ins>
<ins>uniform vec2 distortionTiling = vec2(1., 1.);</ins>
<ins>uniform vec2 distortionOffset = vec2(1., 1.);</ins>
<ins>uniform float distortionStrength = 0.1;</ins>

void fragment()
{
	<ins>vec2 distortionTextureUV = UV * distortionTiling + distortionOffset;</ins>
	<ins>vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;</ins>
	
	COLOR = texture(TEXTURE, UV + distortionNoiseOffset);
}
</pre></td></tr></tbody></table></code></div></div>

Now if we set distortion strength to something like 0.1 and start changing the offset we'll see that the image is shifting.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/distortion_on_image.png">

Let's make it so that the distortion is happening without us having to change the offset in the editor by changing the offset over time.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

uniform sampler2D noiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionOffset = vec2(1., 1.);
uniform float distortionStrength = 0.1;

void fragment()
{
	<del>vec2 distortionTextureUV = UV * distortionTiling + distortionOffset;</del>
	<ins>vec2 distortionTextureUV = UV * distortionTiling + distortionOffset * TIME;</ins>
	<vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;
	
	COLOR = texture(TEXTURE, UV + distortionNoiseOffset);
}
</pre></td></tr></tbody></table></code></div></div>

Now let's get onto integrating it into our dissolve shader. Comment it out for the time being. You will see the image as is.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/default_fox.png">

In the dissolve shader the dissolved part is completely transparent and doesn't have any information about what is behind the object in the scene. To have that information we're going to use SCREEN_TEXTURE

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

uniform sampler2D noiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionOffset = vec2(1., 1.);
uniform float distortionStrength = 0.1;

void fragment()
{
	<ins>/*</ins>vec2 distortionTextureUV = UV * distortionTiling + distortionOffset * TIME;
	<vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;
	COLOR = texture(TEXTURE, UV + distortionNoiseOffset); <ins>*/</ins>
    
	<ins>vec4 screenTexture = texture(SCREEN_TEXTURE, UV);</ins>
	<ins>COLOR = screenTexture;</ins>
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/scr_tex_obj_uv.png">

You can see that currently everything we see on the screen is squished into our sprite and the image is upside down. That's because we're using the object's UV coordinates. Let's use the SCREEN_UV to properly apply the screen texture.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

uniform sampler2D noiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionOffset = vec2(1., 1.);
uniform float distortionStrength = 0.1;

void fragment()
{
	/*vec2 distortionTextureUV = UV * distortionTiling + distortionOffset * TIME;
	<vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;
	COLOR = texture(TEXTURE, UV + distortionNoiseOffset); */
    
	<del>vec4 screenTexture = texture(SCREEN_TEXTURE, UV);</del>
	<ins>vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV);</ins>
	COLOR = screenTexture;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/scr_tex_scr_uv.png">

Looks like our sprite disappeared! What actually happens is that it simply has the same colors as the objects behind it. We can see that if we add some tint to it.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

uniform sampler2D noiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionOffset = vec2(1., 1.);
uniform float distortionStrength = 0.1;
<ins>uniform vec4 invisibilityTint : hint_color;</ins>

void fragment()
{
	/*vec2 distortionTextureUV = UV * distortionTiling + distortionOffset * TIME;
	<vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;
	COLOR = texture(TEXTURE, UV + distortionNoiseOffset); */
    
	vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV);
	<ins>screenTexture += invisibilityTint;</ins>
	COLOR = screenTexture;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/scr_tex_scr_uv_color.png">

We can see the whole quad being tinted instead of the parts that are visible in the original sprite, but it's something that we already dealt with in our dissolve sprite and since we will combine both shaders in the end, this problem will be sorted. Now that we can see our object properly, let's add our calculated distortion to the image. Uncomment the code we had before adn add the following changes: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform vec2 tiling = vec2(1.,1.);
uniform vec2 offset = vec2(0.,0.);

uniform sampler2D noiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionOffset = vec2(1., 1.);
uniform float distortionStrength = 0.1;
<ins>uniform vec4 invisibilityTint : hint_color;</ins>

void fragment()
{
	<del>/*</del>vec2 distortionTextureUV = UV * distortionTiling + distortionOffset * TIME;
	<vec2 distortionNoiseOffset = texture(noiseTexture, distortionTextureUV).rg * distortionStrength;
	<del>COLOR = texture(TEXTURE, UV + distortionNoiseOffset); */</del>
    
	<del>vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV);</del>
	<ins>vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV + distortionNoiseOffset);</ins>
	<ins>screenTexture += invisibilityTint;</ins>
	COLOR = screenTexture;
}
</pre></td></tr></tbody></table></code></div></div>

Now we have all the parts that we need to combine to create the effect, we can use the same method of combining colors as we used to add the colorful edge to our dissolve shader. We already have defined the area that stays visible in the dissolve shader – white parts of the step1 variable.

To get the opposite of it we just need to subtract step1 from 1. We know from the dissolve shader tutorial that white parts of the image are represented by 1 and black parts are represented by 0, subtracting 1 from 1 will give us 0 turning white part into black, and subtracting 0 from 1 will leave us with 1 turning black into white. This way we invert the colors and get the needed area.

Lets combine two shaders into one, make sure to rename the variables to you know what they responsible for. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

<ins>uniform sampler2D dissolveNoiseTexture;</ins>
<ins>uniform float noiseScale = 2;</ins>
<ins>uniform float dissolveAmount : hint_range(0, 1);</ins>
<ins>uniform float edgeThickness = 0.01;</ins>
<ins>uniform vec4 edgeColor : hint_color;</ins>

<del>uniform sampler2D noiseTexture;</del>
<ins>uniform sampler2D distortionNoiseTexture;</ins>
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionSpeed = vec2(1., 1.);
uniform float distortionStrength = 0.1;
uniform vec4 invisibilityTint : hint_color;

void fragment()
{
	<ins>vec4 originalTexture = texture(TEXTURE, UV);</ins>
	<ins>vec2 noiseTextureUV = UV * noiseScale; </ins>
	
	<ins>vec4 dissolveNoise = texture(dissolveNoiseTexture, noiseTextureUV);</ins>

	<ins>vec4 step1 = step(dissolveAmount, dissolveNoise);</ins>
	<ins>vec4 step2 = step(dissolveAmount + edgeThickness, dissolveNoise);</ins>
	
	<ins>vec4 edge = step1 - step2;</ins>
	<ins>vec4 edgeColorArea = edge * edgeColor;</ins> 
	<ins>edgeColorArea.a = originalTexture.a;</ins>
	
	vec2 distortionTextureUV = UV * distortionTiling + TIME * distortionSpeed; 
	vec2 distortionNoiseOffset = texture(distortionNoiseTexture, distortionTextureUV).rg * distortionStrength;
	
	vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV + distortionNoiseOffset);
	screenTexture += invisibilityTint;
    
	<ins>vec4 invertedStep1 = 1.- step1; </ins>
	<ins>vec4 originalAndEdge = mix(originalTexture, edgeColorArea, edge.r);</ins>
	<ins>vec4 combined = mix(originalAndEdge, screenTexture, invertedStep1);</ins>
	
	COLOR = combined;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/combined_no_smoothstep0.1.png">

Looks like it's working, but if we change the strength to something like 0.25 we'll see that everything is shifted quite a lot and it doesn't really look like the object distorts something right behind it.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/combined_no_smoothstep0.25.png">

 That's because the noise texture that we use doesn't have completely black areas, mostly light shades of gray. Which means that when we multiply we increase offset for every part of the image by some amount. To counter that we can either use a texture with more black and darker areas or we could use smoothstep function instead to make our current noise texture darker.

Unlike step funtion it uses two parameters to define limits - one lower limit (values lower that this limit will be 0), and one upper limit (values higher than this limit will be 1). Values in between will smoothly go from 0 to 1 without the hard edge like in the step function.

You can check out the graph for it here

[https://www.desmos.com/calculator/xykhidbkbg](https://www.desmos.com/calculator/xykhidbkbg)

This is the last change we need to complete this shader. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform sampler2D dissolveNoiseTexture;
uniform float noiseScale = 2;
uniform float dissolveAmount : hint_range(0, 1);
uniform float edgeThickness = 0.01;
uniform vec4 edgeColor : hint_color;
uniform vec4 invisibilityTint : hint_color;

uniform sampler2D distortionNoiseTexture;
uniform vec2 distortionTiling = vec2(1., 1.);
uniform vec2 distortionSpeed = vec2(1., 1.);
uniform float distortionStrength = 0.1;

void fragment()
{
	vec4 originalTexture = texture(TEXTURE, UV);
	vec2 noiseTextureUV = UV * noiseScale; 
	
	vec4 dissolveNoise = texture(dissolveNoiseTexture, noiseTextureUV);

	vec4 step1 = step(dissolveAmount, dissolveNoise); 
	vec4 step2 = step(dissolveAmount + edgeThickness, dissolveNoise);
	
	vec4 edge = step1 - step2; 
	vec4 edgeColorArea = edge * edgeColor; 
	
	edgeColorArea.a = originalTexture.a; 
	
	vec2 distortionTextureUV = UV * distortionTiling + TIME*distortionSpeed; 
	vec2 distortionNoiseOffset = texture(distortionNoiseTexture, distortionTextureUV).rg * distortionStrength;
	
	<ins>distortionNoiseOffset = smoothstep(vec2(0.1), vec2(1.,1.), distortionNoiseOffset);</ins>
	
	vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV + distortionNoiseOffset);
	screenTexture += invisibilityTint;
	vec4 invertedStep1 = 1.- step1; 

	vec4 originalAndEdge = mix(originalTexture, edgeColorArea, edge.r); 
	vec4 combined = mix(originalAndEdge, screenTexture, invertedStep1);
	
	COLOR = combined;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotInvisibilityShaderTutorial/Invisibility_final.gif">

# Notes

For this shader to work on 3d object you'll need following changes: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type spatial;

<ins>uniform sampler2D baseTexture;</ins>
uniform sampler2D dissolveNoiseTexture;
uniform float noiseScale = 2;
uniform float dissolveAmount : hint_range(0, 1);
uniform float edgeThickness = 0.01;
uniform vec4 edgeColor : hint_color;
uniform vec4 invisibilityTint : hint_color;

uniform sampler2D distortionNoiseTexture;
uniform vec2 distortionScale = vec2(1., 1.);
uniform vec2 distortionSpeed = vec2(1., 1.);
uniform float distortionStrength = 0.1;

void fragment()
{
	<del>vec4 originalTexture = texture(TEXTURE, UV);</del>
	<ins>vec4 originalTexture = texture(baseTexture, UV);</ins>
	vec2 noiseTextureUV = UV * noiseScale; 
	
	vec4 dissolveNoise = texture(dissolveNoiseTexture, noiseTextureUV);

	vec4 step1 = step(dissolveAmount, dissolveNoise); 
	vec4 step2 = step(dissolveAmount + edgeThickness, dissolveNoise);
	
	vec4 edge = step1 - step2; // difference between results of two step functions is the edge area
	vec4 edgeColorArea = edge * edgeColor; 
	
	edgeColorArea.a = originalTexture.a; 
	
	vec2 distortionTextureUV = UV * distortionScale + TIME*distortionSpeed; 
	vec2 distortionNoiseOffset = texture(distortionNoiseTexture, distortionTextureUV).rg * distortionStrength;
	
	distortionNoiseOffset = smoothstep(vec2(0.1), vec2(1.,1.), distortionNoiseOffset);
	
	vec4 screenTexture = texture(SCREEN_TEXTURE, SCREEN_UV + distortionNoiseOffset);
	screenTexture += invisibilityTint;
	vec4 invertedStep1 = 1.- step1;

	vec4 originalAndEdge = mix(originalTexture, edgeColorArea, edge.r); 
	vec4 combined = mix(originalAndEdge, screenTexture, invertedStep1);
	<del>COLOR = combined;</del>
	<ins>ALBEDO = combined.rgb;</ins>
	<ins>EMISSION = combined.rgb;</ins>
}
</pre></td></tr></tbody></table></code></div></div>