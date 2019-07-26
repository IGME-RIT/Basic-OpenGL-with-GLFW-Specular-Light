Documentation Author: Niko Procopi 2019

This tutorial was designed for Visual Studio 2017 / 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Specular Light Tutorial!
Prerequesites: Draw a cube with a texture and a point-light

Disclaimer:
This tutorial builds off the tutorial with normal mapping
and anisotropic filtering. An understanding of these topics
are not required for Specular Lighting.

Comparison Screenshot:
Left:   texture + normal + anisotropy
Middle: texture + normal + anisotropy + specular lighting
Right:  texture + normal + anisotropy + specular mapping

We changed the textures from brick to stone, because the 
Specular map for the stone material made more of a visible
difference than the brick material (used in the tutorial
after this)

What is Specular Lighting:
In real life, if you go to a pond in the middle of the day,
most of the water looks blue, but then you can see the reflection
of the sunlight in the water, which shows up as white.
Diffuse light is the reflection of "light" hitting your eye
"light" is any form of light rays that reflect off objects
Specular light is the reflection of "a light" hitting your eye
"a light" meaning the light source, like the sun, or a lightbulb

How it works:
When calculating the color for each individual pixel,
we need to get the direction from the pixel, to the light,
then we need to reflect that direction on an axis (like the x, y, or z axis).
The axis that we need to reflect the direction on, is the normal.
Sounds confusing, but think about it, imagine what direction the normal points in,
reflecting the direction FROM pixel TO light over the normal, is an accurate reflection
of a light ray. Regardless if we are using an interpolated vertex normal, or if we are
using normals from a normal map, it works the same way.
The direction FROM pixel TO light, reflected, is the direction that the light reflected in
Next, we need to get the direction FROM the pixel TO the camera.
If the reflected light's direction is the same as the direction FROM the pixel TO the camera,
then it means the a light (sun, lightbulb) reflected off a surface and hit the camera

How it works (part 2):
Remember how we use NdotL for diffuse light?
If a vertex normal points straight to a light, there is bright diffuse light
There is a direction for vertex normal, and direction FROM point TO light,
as those direction gets farther apart, the diffuse light gets darker
Specular light is the same thing
We have two directions: reflected ligiht, and direction FROM pixel TO camera
The closer the vectors are together, the brighter the light, the farther apart,
the darker the light, so we use another dot product
After we have this dot product, we multiply it by the color of the light (RGB),
then we add (lightColor * newDotProduct) to gl_FragColor, and we're done

How to implement (Part 1):
In main.cpp
Prepare to send the camera position, and specular exponent to shader
	char cameraPositionFS[] = "cameraPosition";
	char specularExponentFS[] = "specularExponent";
The specular exponent will control how shiny the overall surface is.
Next, send the values to the shader
        material->SetVec3(cameraPositionFS, controller.GetTransform().Position());
        material->SetInt(specularExponentFS, specExp);
We get the position from the camera's Transform, and then we get specularity from specExp.
specExp can be adjusted with the 'Q' and 'E' buttons while the program is running

How to implement (Part 2):
In fragment.glsl
We need to import the Camera position and Specular Exponent as uniforms
	uniform vec3 cameraPosition;
	uniform int specularExponent;
We hardcode values for the light
AmbientLight is the darkest color that a pixel can be
Position and color... well... I hope you know what those are
Attenuation is the rate that the light dies as distance from light increases
This method of attenuation is a cheap way to fake a spot-light
Range is the radius of the light
	vec4 ambientLight = vec4(.1, .1, .3, 1);
	vec3 pointLightPosition = vec3(-2, 1, 1);
	vec4 pointLightColor = vec4(1, .8, .3, 1);
	vec3 pointLightAttenuation = vec3(3, 1, 0);
	float pointLightRange = 5;
We then get attenuation (0 to 1) with this formula:
	float attenuation = 1 / (distance * distance * pointLightAttenuation.x + distance * pointLightAttenuation.y + pointLightAttenuation.z);
The final light color is multiplied by attenuation to make the light get darker as distance increases.
By looking at previous tutorials, this should be familiar information.
In previous tutorials, we got the direction FROM pixel TO light when we did Diffuse
	vec3 lightDir = pointLightPosition - position;
Now, we need to reflect this direction, which is easier than it sounds
	vec3 reflectDir = normalize(reflect(-lightDir, norm));
The "normal" that we reflect the direction over, can either be a rasterized vertex-normal, or 
a normal from the normal map. If normal mapping is already implemented, always use the 
newly calculated normal (with TBN) instead of the original vertex normal
Get the direction FROM pixel TO camera
	vec3 surfaceToEyeDir = normalize(vec3(cameraPosition) - position);
Use the Dot product to find the brightness of the light (as described in "how it works"):
	float specularLightBrightness = dot(reflectDir, surfaceToEyeDir);
Find the specular light color with this formula:
	// first we clamp: clamp(specularLight, 0.0, 1.0)
	// then we use the exponent (from C++): pow( ..., specularExponent)
	// then we multiply by attenuation
	specularLight = pow(clamp(specularLight, 0.0, 1.0), specularExponent) * attenuation;
Find the final specular light color by multiplying specular light by the RGB color that you want:
	vec4 finalSpecularColor = pointLightColor * specularLight;
Finally, add the specular color to the final pixel color on the screen:
	gl_FragColor = (color * finalDiffuseColor) + finalSpecularColor;

How to improve:
Try Specular mapping (next tutorial) to improve the quality of specular light
Try passing light data (position, color) as uniforms, so that they can change each frame
	Then you can make the light gradually change position, color, and size