# UV coordinate setup

```GLSL
vec2 uv = (2.*gl_FragCoord.xy-resolution.xy)/resolution.x;
```

Given an display aspect ratio of `ar = resolution.x/resolution.y` it returns coordinates in: x ∈ [-1,1] ⊗ y ∈ [-1/ar,1/ar].

# Basic raytracer

```GLSL
bool raytrace(vec3 ro, vec3 rd, out vec3 p) {
	float f = 0.;
	
	const float FMAX = 10.;
	
	for (int i=0; i<256; i++) {
		p = ro+f*rd;
		float d = scene(p);
		if (d <= EPS) return true;
		if (f > FMAX) return false;
		f += d;
	}
	
	return false;
}
```

Returns if the casted ray with origin `ro` and direction `rd` hit an object defined in the `scene()` function. `p` returns the `vec3` hit position with `FMAX` denoting the furthest distance to probe and `EPS` the resolution of declaring a hit.

## Camera setup

```GLSL
mat3 camera(vec3 ro, vec3 t, float cr) {
	vec3 cw = normalize(t-ro);
	vec3 cp = vec3(sin(cr), cos(cr),0.0);
	vec3 cu = normalize(cross(cw,cp));
	vec3 cv = normalize(cross(cu,cw));
    return mat3(cu, cv, cw);
}
```

Returns a `mat3` to be multiplied with the ray direction `ro` (typically `vec3 rd = normalized(vec3(uv.xy, 1.));`) to get the proper ray direction for raymarching. The given `vec3 t` defines the lookat target and the 


# Normal calculation

```GLSL
vec3 normal(vec3 p) {
	vec2 e = vec2(EPS, 0.);
	float d = scene(p);
	return normalize(vec3(d-scene(p+e.xyy), d-scene(p+e.yxy), d-scene(p+e.yyx)));
}
```

Calculates the normal vector at the given position `p` by approximating the directional derivatives in x, y and z.

# Diffuse lighting

```GLSL
vec3 lighting(vec3 p, vec3 lightPos) {
	vec3 n = normal(p);
	vec3 lightDir = normalize(p-lightPos);
	vec3 color = vec3(max(dot(lightDir,n), 0.));
	return color;
}
```

Returns a grayscale value of the diffuse lighting of a given position `p` with a light source at `lightPos`.

# Ambient occlusion

```GLSL
float calcAO(vec3 p, vec3 n) {
	float occ = 0., sca = 1.;
	for (int i=0; i<5; i++) {
		float hr = .01 + .12*float(i)/4.;
		vec3 aopos = p + n * hr;
		float d = scene(aopos);
		occ += -(d-hr)*sca;
		sca *= .95;
	}
	return clamp(1.-3.*occ, 0., 1.);
}
```

Returns a float ∈ [0,1] to be used as a prefactor for lighting.

# Distance fields

A good reference for different distance field functions and operators is given by iq on his [homepage](http://iquilezles.org/www/articles/distfunctions/distfunctions.htm) and by mercury on their [homepage](http://mercury.sexy/hg_sdf/).
