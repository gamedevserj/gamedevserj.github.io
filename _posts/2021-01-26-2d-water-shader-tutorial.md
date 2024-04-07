---
layout: post
title:  "Water shader tutorial"
date: 2021-01-26
tags: tutorial shader
author: gamedevserj
---

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/Water_final.gif" alt="Final gif">

The shader contains the following features: distortion, reflection, shoreline foam, and waves. We already covered distortion in the previous tutorial about invisibility shader, so all we need is reflection, shoreline, and waves.

A couple of notes before we start:

 - With this setup distance between water sprite and top of the screen must be at least sprite's height or more, otherwise there's no information to reflect.
 - For proper reflection pivot should be at the top of the sprite, so set the sprite's offset on Y-axis accordingly. 

Let's start with waves. Our current water is going to be a white sprite, so that we can see changes that we make. We're going to define a variable called distFromTop that controls distance from the top of the sprite that needs to be transparent. For now it's going to be a simple float. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	float distFromTop = 0.0;
	float waveArea = UV.y - distFromTop;

	clr.a *= waveArea;
	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/start.png" alt="Initial image">

If you change the variable to something like 0.5 you'll see that top part of the sprtie becomes lighter. This is how we're going to make waves – by changing the alpha of a certain chuck at the top. 

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/dist_0.5.png" alt="Distance 0.5">

Let's add a smoothstep method to make things more distinct and allow us to control the "smoothness" of our waves.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float waveSmoothing = .1;

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	float distFromTop = 0.5;
	float waveArea = UV.y – distFromTop;
	
	<ins>waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);</ins>
	color.a *= waveArea;
	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/smoothstep.png" alt="Smoothstep">

Now that we have that setup we can calculate the distFromTop variable to make actual waves. For that we're going to use sine wave function.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

<ins>uniform float waveSmoothing = .1;</ins>
<ins>uniform float mainWaveSpeed = 2.5;</ins>
<ins>uniform float mainWaveFrequency = 20;</ins>
<ins>uniform float mainWaveAmplitude = 0.005;</ins>

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	<del>float distFromTop = 0.5;</del>
	<ins>float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed);</ins>
	float waveArea = UV.y – distFromTop;
	
	waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);
	color.a *= waveArea;
	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

It looks fine when our amplitude is quite low, but if we set it to higher values the top of the waves if going to be cut off.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/wave_cut_off.png" alt="Wave cut off">

To solve that we simply add wave amplitude at the end of our equation.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float waveSmoothing = .1;
uniform float mainWaveSpeed = 2.5;
uniform float mainWaveFrequency = 20;
uniform float mainWaveAmplitude = 0.005;

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	<del>float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed);</del>
	<ins>float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed) + mainWaveAmplitude;</ins>
	float waveArea = UV.y – distFromTop;
	
	waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);
	color.a *= waveArea;
	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/wave_fixed.png" alt="Wave fixed">

You notice that variables have 'main' prefix and our waves don't look natural, by adding another set of waves we can achieve a more randomized look. The code in the repository will have those waves, but we're going to skip this part here. Next one is shoreline, it's quite simple – we're going to use the same calculation we did for the wave area, but we're going to subtract a value – which will determine our shoreline size. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float waveSmoothing = .1;
uniform float mainWaveSpeed = 2.5;
uniform float mainWaveFrequency = 20;
uniform float mainWaveAmplitude = 0.005;

<ins>uniform vec4 shorelineColor : hint_color = vec4(1.);</ins>
<ins>uniform float shorelineSize : hint_range(0., 1.) = 0.0;</ins>

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed) + mainWaveAmplitude;
	float waveArea = UV.y – distFromTop;
	
	waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);
	color.a *= waveArea;

	<ins>float shorelineBottom = UV.y - distFromTop - shorelineSize;</ins>
	<ins>shorelineBottom = smoothstep(0., 1. * waveSmoothing,  shorelineBottom);</ins>
	<ins>float shoreline = waveArea - shorelineBottom;</ins>
	<ins>color.rgb += shoreline * shorelineColor.rgb;</ins>

	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/shoreline.png" alt="Shoreline">

Additionally you might want to have a shoreline foam, which is quite simple to add. The calculation is similat to the one we use for waveArea.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float waveSmoothing = .1;
uniform float mainWaveSpeed = 2.5;
uniform float mainWaveFrequency = 20;
uniform float mainWaveAmplitude = 0.005;

uniform vec4 shorelineColor : hint_color = vec4(1.);
uniform float shorelineSize : hint_range(0., 1.) = 0.0;
<ins>uniform float foamSize : hint_range(0., 1.0) = 0.0025;</ins>
<ins>uniform float sfoamStrength : hint_range(0., 1.0) = 0.5;</ins>
<ins>uniform float foamSpeed;</ins>
<ins>uniform vec2 foamScale;</ins>

void fragment()
{
	vec4 color = vec4(0.,0.,0.,1.);
	float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed) + mainWaveAmplitude;
	float waveArea = UV.y – distFromTop;

	waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);
	color.a *= waveArea;

	float shorelineBottom = UV.y - distFromTop - shorelineSize;
	shorelineBottom = smoothstep(0., 1. * waveSmoothing,  shorelineBottom);
	
	float shoreline = waveArea - shorelineBottom;
	color.rgb += shoreline * shorelineColor.rgb;

	<ins>vec4 foamNoise = texture(noiseTexture, UV* foamScale + TIME * foamSpeed);</ins>
	<ins>foamNoise.r = smoothstep(0.0, foamNoise.r, foamStrength); </ins>

	<ins>float shorelineFoam = UV.y – distFromTop;	</ins>
	<ins>shorelineFoam = smoothstep(0.0, shorelineFoam, foamSize);</ins>

	<ins>shorelineFoam *= foamNoise.r;</ins>
	<ins>color.rgb += shorelineFoam * shorelineColor.rgb;</ins>

	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/shoreline_foam.png" alt="Shoreline foam">

Note that we can calculate the shoreline in the same manner and it would blend in better into foam, if that's the look you're going after.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/shoreline_foam_blending.png" alt="Shoreline foam blending">

The last part to add is the reflection. If you've read the magnifying glass tutorial or invisibility effect shader tutorial you probably know that we're going to use SCREEN_TEXTURE. And similarly to magnifying glass shader tutorial most of the work is going to be done in script. Since we're flipping the SCREE_TEXTURE vertically around the center of the screen we're going to need our object to track it's position on screen and adjust the offset accordingly. Here's a visual representation of the issue. In the first image the water's origin is in the middle of the screen and reflection works fine.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/reflection_centered.png" alt="Reflection centered">

In the second image water is located lower than the screen center, left side is original and right side is flipped. You can see that top of the house matches the reflection (apart from the wave distortion), this means that we need to offset the reflection.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotWater2DShaderTutorial/reflection_issue.png" alt="Reflection issue">

Here's the script that takes into account position of the water:

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
extends Node

export(NodePath) var cameraPath

var scrHeight;
var calculatedOffset : float;

func _ready():
	scrHeight = ProjectSettings.get_setting("display/window/size/height");

func _process(delta):
	var camZoom = get_node(cameraPath).zoom.y;
	
	calculatedOffset = (-get_node(cameraPath).position.y/(scrHeight) + self.position.y/scrHeight) * 2 / camZoom;
	
	self.material.set_shader_param("calculatedOffset", calculatedOffset);
</pre></td></tr></tbody></table></code></div></div>

Water distortion is achieved the same way to the [invisibility effect shader]({% post_url 2020-09-16-invisibility-effect-shader-tutorial %}).

Here's the final shader code:

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float reflectionBlur = 0; // works only if project's driver is set to GLES3, more information here https://docs.godotengine.org/ru/stable/tutorials/shading/screen-reading_shaders.html
uniform float calculatedOffset = 0; // this is controlled by script, it takes into account camera position and water object position, that way reflection stays in the same place when camera is moving
uniform sampler2D noiseTexture;

uniform vec2 distortionScale = vec2(0.3, 0.3);
uniform vec2 distortionSpeed = vec2(0.01, 0.02);
uniform vec2 distortionStrength = vec2(0.3, 0.3);

uniform float waveSmoothing = .01;

uniform float mainWaveSpeed = 2.5;
uniform float mainWaveFrequency = 20;
uniform float mainWaveAmplitude = 0.005;

uniform vec4 shorelineColor : hint_color = vec4(1.);
uniform float shorelineSize : hint_range(0., 1.) = 0.0025;
uniform float foamSize : hint_range(0., 1.0) = 0.0025;
uniform float foamStrength : hint_range(0., 1.0) = 0.5;
uniform float foamSpeed;
uniform vec2 foamScale;

void fragment()
{
	
	vec2 uv = SCREEN_UV; 
	uv.y = 1. - uv.y; // turning screen uvs upside down
	uv.y -= calculatedOffset;
	
	vec2 noiseTextureUV = UV * distortionScale; 
	noiseTextureUV += TIME * distortionSpeed; // scroll noise over time
	
	vec2 waterDistortion = texture(noiseTexture, noiseTextureUV).rg;
	waterDistortion.rg *= distortionStrength.xy;
	waterDistortion.rg = smoothstep(0.0, 1., waterDistortion.rg); 
	uv += waterDistortion;
	
	vec4 color = textureLod(SCREEN_TEXTURE, uv, reflectionBlur);

	float distFromTop = mainWaveAmplitude * sin(UV.x * mainWaveFrequency + TIME * mainWaveSpeed) + mainWaveAmplitude;
	
	float waveArea = UV.y - distFromTop;
	waveArea = smoothstep(0., 1. * waveSmoothing, waveArea);
	
	color.a *= waveArea;

	float shorelineBottom = UV.y - distFromTop - shorelineSize;
	shorelineBottom = smoothstep(0., 1. * waveSmoothing,  shorelineBottom);
	
	float shoreline = waveArea - shorelineBottom;
	color.rgb += shoreline * shorelineColor.rgb;
	
	vec4 foamNoise = texture(noiseTexture, UV* foamScale + TIME * foamSpeed);
	foamNoise.r = smoothstep(0.0, foamNoise.r, foamStrength); 
	
	float shorelineFoam = UV.y - distFromTop;
	shorelineFoam = smoothstep(0.0, shorelineFoam, foamSize);
	
	shorelineFoam *= foamNoise.r;
	color.rgb += shorelineFoam * shorelineColor.rgb;
	
	COLOR = color;
}
</pre></td></tr></tbody></table></code></div></div>
