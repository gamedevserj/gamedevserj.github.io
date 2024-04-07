---
layout: post
title:  "Fade out shader tutorial"
date: 2022-06-05
tags: tutorial shader
---

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/fade_out_final.gif" alt="Final gif">

Today we're going to be figuring out how to make a fade out shader that can fade out to a certain point of the screen.

I'm assuming you've read previous tutorials and know basics by now. Create a new shader, apply it to the material, create a sprite in your scene, and add that material to the sprite object.

Let's start by creating some parameteres that are going to control the way fade out works and looks. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform vec4 fadeColor : hint_color;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

You can use any image you want for the sprite object, with the code above it should turn into a square with the color defined by fadeColor parameter. In this tutorial I will be using a 512x512 image and will set the fade out color to be black. If you're following the tutorial, make sure you're using the image with the equal width and height.

We're going to be using UV coordinates to change how much of the object is faded out, so let's add that to our code and see how it looks. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform vec4 fadeColor : hint_color;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	<ins>vec2 center = UV;</ins>
	<ins>float fadeDistance = length(center);</ins>
	<ins>fadeMask.a = fadeDistance;</ins>
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade.jpg" alt="UV fade">

As you can see the upper left corner is completely transparent, while the bottom right one is completely black. That's because as we already know our UVs start at the tope left. Let's add a step function to see the difference more clearly.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform vec4 fadeColor : hint_color;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV;
	float fadeDistance = length(center);
	
	<del>fadeMask.a = fadeDistance;</del>
	<ins>fadeMask.a = step(fadeDistance, fadeAmount);</ins>
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_step.jpg" alt="UV fade step">

Now, if we set fadeAmount to 1 we will see how it affects our image. The length() function returns the magnitude of the vector we provided for it – in this case UV coordinates. The bottom right corner has coordinates of (1, 1) and magnitude of such vector is ~1.4142, that's why at the fadeAmount of 1 we still have a corner that is completely transparent.

It is an issue that we'll sort out later, but right now we should set our point of fade to be the center of the image. And it is quite easy to do. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform vec4 fadeColor : hint_color;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	<del>vec2 center = UV;</del>
	<ins>vec2 center = UV - 0.5;</ins>
	float fadeDistance = length(center);
	
	fadeMask.a = step(fadeDistance, fadeAmount);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_center.jpg" alt="UV fade center">

We simply offset our UV coordinates by 0.5 and there you go – we have a circle! You'll notice that the effect we have is inverted – instead of having a transparent circle and blacked out sides we have a black circle, don't worry we address it a few steps later.

Now if you play around with the fadeAmount parameter you will see that it fades out the image completely at ~0.707. Why is that? Let's break it down in detail.

Unmodified UVs are as follows:
Top left (0, 0)
Top right (1, 0)
Bottom left (0, 1)
Bottom right (1, 1)

By subtracting 0.5 we changed it to:
Top left (-0.5, -0.5)
Top right (0.5, -0.5)
Bottom left (-0.5, 0.5)
Bottom right (0.5, 0.5)

And if we use those coordinates as vectors and get their length we will find out that it is equal to ~0.707. How can we check it? Well we can use the Pythagorean theorem – look at the image below and see that in the case of corners calculating the length of a vector is essentially calculating the hypotenuse of a right angle triangle with equal sides.

Wonderful, now that we know this let's create a parameter for it and remap the fade out value so that the complete fade out happened when fadeAmount reaches 1. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform vec4 fadeColor : hint_color;
<ins>uniform float hypotenuse = 0.707;</ins>

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV - 0.5;
	float fadeDistance = length(center);
	<ins>float remappedFade = fadeAmount * hypotenuse;</ins>
	
	fadeMask.a = step(fadeDistance, remappedFade);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

Looks good! But what if we wanted to have a smoother transition from visible to invisible parts? Let's add a fadeSmoothing parameter and replace the step() function with smoothstep().

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV - 0.5;
	float fadeDistance = length(center);
	float remappedFade = fadeAmount * hypotenuse;
	
	<del>fadeMask.a = step(fadeDistance, remappedFade);</del>
	<ins>fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, remappedFade);</ins>
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_smoothstep_broken.jpg" alt="UV fade smoothstep broken">

Hmm, I think we broke something – when fadeSmoothing is set to 0 the image fades out instantly, if we increase and and play around with fadeAmount parameter it fades out gradually, but it does so equally in every part of the image. The reason is previously we compared fadeAmount to the fadeDistance, the area where fadeDistance was greater than fadeAmount was transparent. And in smoothstep() function we just added this value doesn't take part at all. Let's add it in.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV - 0.5;
	float fadeDistance = length(center);
	float remappedFade = fadeAmount * hypotenuse;
	
	<del>fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, remappedFade);</del>
	<ins>fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, fadeDistance - remappedFade);</ins>
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_smoothstep_fixed.jpg" alt="UV fade smoothstep fixed">

What does exactly happens after we added this subtraction? Now we're comparing the difference between the distance and our fadeAmount(remapped) which inverted our effect – exactly what we needed. There's one small thing – when we set our fadeAmount to 1 it is completely transparent. I prefer it to be opposite – transparent image when fadeAmount is 0 and completely faded out when it is 1. It is a simple fix, we just need to invert the fadeAmount by subtracting it from 1.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV - 0.5;
	float fadeDistance = length(center);
	<ins>float invertedFadeAmount = 1.0 - fadeAmount;</ins>
	
	<del>float remappedFade = fadeAmount * hypotenuse;</del>
	<ins>float remappedFade = invertedFadeAmount * hypotenuse;</ins>
	
	fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, fadeDistance - remappedFade);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

Now it works like we want it to. But there's one issue – if we set fadeSmoothing to something more than 0, say 0.2 and increase our fadeAmount to 1 you'll see that the image is not completely faded out. That's because we don't account for the smoothing in our calculations of remappedFade and the value of the fadeOut is not greater than the upper limit of the smoothstep() function. Let's fix that.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	vec2 center = UV - 0.5;
	float fadeDistance = length(center);
	float invertedFadeAmount = 1.0 - fadeAmount;
	
	<del>float remappedFade = invertedFadeAmount * hypotenuse;</del>
	<ins>float remappedFade = invertedFadeAmount * (invertedFadeAmount * hypotenuse + fadeSmoothing) - fadeSmoothing;</ins>
	
	fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, fadeDistance - remappedFade);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

Look at that, we have a fade out shader! So, now we need to create a parameter that will control where exactly our circle is centered at. Judging by variable names it should be the one we called center, and we can modify it by subtracting some offset. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;
<ins>uniform vec2 offset = vec2(0.5, 0.5);</ins>

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	<del>vec2 center = UV - 0.5;</del>
	<ins>vec2 center = UV - offset;</ins>
	float fadeDistance = length(center);
	float invertedFadeAmount = 1.0 - fadeAmount;
	
	float remappedFade = invertedFadeAmount * (invertedFadeAmount * hypotenuse + fadeSmoothing) - fadeSmoothing;
    
	fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, fadeDistance - remappedFade);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

Now when our offset.x value is equal to 1 the center is at the right point of the image and if it is 0 – at the left side. Similarly for the Y-axis where 1 is the bottom and 0 is the top.

<img class = "image-in-tutorial" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_right_side.jpg" alt="UV fade right side">

Is that it? Do we finally have a fully working shader? Nope.

Usually compters have screens that are not square shaped, but if we scale our sprite, say make it 2 times wider we'll see that our perfect circle turns into an oval. 

<img class = "image-in-tutorial-wide" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_oval.jpg" alt="UV fade oval">

So, to fix that we'll add a tiling parameter and multiply the UV by it.

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
shader_type canvas_item;

uniform float fadeAmount : hint_range(0, 1) = 1;
uniform float fadeSmoothing : hint_range(0, 1) = 0.5;
uniform vec4 fadeColor : hint_color;
uniform float hypotenuse = 0.707;
uniform vec2 offset = vec2(0.5, 0.5);
<ins>uniform vec2 tiling = vec2(1.0, 1.0);</ins>

void fragment()
{
	vec4 fadeMask = fadeColor;
	
	<del>vec2 center = UV - offset;</del>
	<ins>vec2 center = UV * tiling - offset;</ins>
	
	float fadeDistance = length(center);
	
	float invertedFadeAmount = 1.0 - fadeAmount;
	
	float remappedFade = invertedFadeAmount * (invertedFadeAmount * hypotenuse + fadeSmoothing) - fadeSmoothing;
	
	fadeMask.a = smoothstep(0.0, 1.0 * fadeSmoothing, fadeDistance - remappedFade);
	
	COLOR = fadeMask;
}
</pre></td></tr></tbody></table></code></div></div>

Lets set the X value of the tiling parameter to be 2. The circle is back, but it is moved to the left side. Why is that? 

<img class = "image-in-tutorial-wide" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_oval_fixed_left_side.jpg" alt="UV fade oval fixed left side">

Previously our UVs went from 0 to 1:
Top left (0, 0)
Top right (1, 0)
Bottom left (0, 1)
Bottom right (1, 1) A

nd after subtracting the offset were as follows:
Top left (-0.5, -0.5)
Top right (0.5, -0.5)
Bottom left (-0.5, 0.5)
Bottom right (0.5, 0.5)

Now let's see what the corner coordinates look like when we multiplied the UVs, but before we subtracted the offset
Top left (0, 0)
Top right (2, 0)
Bottom left (0, 2)
Bottom right (2, 2)

And after we subtract the offset of 0.5
Top left (-0.5, -0.5)
Top right (1.5, -0.5)
Bottom left (-0.5, 1.5)
Bottom right (1.5, 1.5)

So the center is moved and if we set the offset.x to be equal to 1 it's going to be centered again. But it doesn't fade out the image completely – the sides are still black. Why is that? Because our hypotenuse is no longer equal to ~0.707 because the length of our triangle side on the X-axis is different.

<img class = "image-in-tutorial-wide" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_wide_not_full.jpg" alt="UV fade wide not full">

So how do we deal with this? We can make our fade object square again and increase its size to be bigger than the screen and then move it to the point where we want the fade to collapse at. Nothing wrong with this approach, just one small thing – say, for example, we want the center of the fade out to be right at one of the edges of the screen. That means that our object needs to have the two times the width of the screen. But if we have the object of that size and our fade out center is in the center of the screen it would mean that from the moment fade out starts, to the moment when it is visible on screen there will be some time where nothing is happening. See the image below to visualize the issue. 

<img class = "image-in-tutorial-wide" src="https://raw.githubusercontent.com/gamedevserj/Images-For-Repo/main/Site/GodotFadeOutShaderTutorial/uv_fade_size_issue.jpg" alt="UV fade size issue">

It's not a huge thing, but we can deal with it. We can either do it by adjusting the size of the sprite object when we determine the point where the fade out should happen, or we can set the image to be the size of the screen and simply adjust shader parameters.

I'll post the complete script and do a short explanation. 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>
extends Node2D

export(float) var fadeDuration = 0.5;
export(float) var fadeAmount = 0.0;

var targetPosition = Vector2(0.0, 0.0);

var aspect = float(1.0);
var tiling = Vector2(1.0, 1.0);
var offset = Vector2(0.0, 0.0);
var hypotenuse = float(0.707);

func _ready():
	_calculateAspect();
	_calculateOffset();
	_calculateHypotenuse();
	pass 

func _process(delta):
	_setShaderParameters();
	pass

func _input(event):
	if event is InputEventMouseButton:
		if event.button_index == BUTTON_LEFT and event.pressed:
			targetPosition = _normalizedScreenPosition(event.position);
			_fadeOut();

func _normalizedScreenPosition(screenPos):
	screenPos.x /= OS.window_size.x;
	screenPos.y /= OS.window_size.y;
	return screenPos;
	
func _fadeOut():
	_calculateOffset();
	_calculateHypotenuse();
	var tween = self.get_node("Tween");
	tween.interpolate_property(self, "fadeAmount", 0, 1, fadeDuration, Tween.TRANS_LINEAR);
	tween.interpolate_callback(self, fadeDuration + 0.1, "_fadeIn");
	tween.start();
	pass
	
func _fadeIn():
	var tween = self.get_node("Tween");
	tween.interpolate_property(self, "fadeAmount", 1, 0, fadeDuration, Tween.TRANS_LINEAR);
	tween.start();
	pass

func _calculateAspect():
	var size = OS.window_size;
	if(size.x > size.y):
		aspect = size.x / size.y;
		tiling.x = aspect;
		tiling.y = 1;
	else:
		aspect = size.y / size.x;
		tiling.y = aspect;
		tiling.x = 1;
	pass

func _calculateOffset():	
	offset = targetPosition;
	
	if(OS.window_size.x > OS.window_size.y):
		offset.x *= aspect;
	else:
		offset.y *= aspect;
	pass

func _calculateHypotenuse():
	var x = offset.x;
	var y = offset.y;
	
	var screenCenterNormalized = Vector2(0.5, 0.5);
	if(OS.window_size.x > OS.window_size.y):
		screenCenterNormalized.x *= aspect;
	else:
		screenCenterNormalized.y *= aspect;
	
	# fade out center is in the left portion of the screen, calculate distance to the right side of the screen
	if(x < screenCenterNormalized.x):
		x = screenCenterNormalized.x * 2  - x;
	# fade out center is in the bottom portion of the screen, calculate distance to the top side of the screen
	if(y < screenCenterNormalized.y):
		y = screenCenterNormalized.y * 2  - y;
	
	hypotenuse = sqrt(x * x + y * y);
	pass

func _setShaderParameters():
	self.material.set_shader_param("fadeAmount", fadeAmount);
	self.material.set_shader_param("hypotenuse", hypotenuse);
	self.material.set_shader_param("offset", offset);
	self.material.set_shader_param("tiling", tiling);
	pass
</pre></td></tr></tbody></table></code></div></div>

_fadeIn() and _fadeOut() funcitons cimply tween the fade amount.

_getNormalizedScreenPosition() is self-explanatory. We save the normalized position in the targetPosition variable.

_calculateAspect() is where we calculate the aspect ratio of our screen, since it is what is going to determine our tiling and affect the offset. We simply check which is greater – the width or the height, and then divide the greater value by the smaller one. So in the screen with resolution of 1920x1080 the aspect ratio is going to be 1.7777.

_calculateOffset() is self-explanatory as well, we just multiply the offset value to center our fade out around the targetPosition. It's the same thing that we did after we increased the width of our sprite by 2, set the tiling.x to 2, and then increased offset.x by 2 as well.

_calculateHypotenuse() is the most interesting one of the bunch. We check where the fade out center point is located. If it's in the left part of the screen – the offset.x value is going to be less than half of the normalied screen width, so we need to calculate the distance to the right side. And if it is at the top – we calculate distance to the bottom. And after that we can calculate the hypotenuse that will determine the outer edge of our fade out.

And that's it! All you need to do is setup your object to be the size of your screen and you can fade in/out to any point you want! 