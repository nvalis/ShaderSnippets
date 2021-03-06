# UV coordinate setup

```GLSL
vec2 uv = (2.*gl_FragCoord.xy-resolution.xy)/resolution.x;
```

Given an display aspect ratio of `ar = resolution.x/resolution.y` it returns coordinates in: x ∈ [-1,1] ⊗ y ∈ [-1/ar,1/ar].

# Basic raytracer

```GLSL
float refd = 1./0.;
void raytrace(vec3 ro, vec3 rd, out vec3 p) {
	p = ro;
	float d;
	vec3 n;
	for (int i=0; i<256; i++) { 
		p += rd*(d=scene(p));
		if (d<EPS) {
			if(refd==d) {
				p += EPS*(n=normal(p));
				rd = reflect(rd, n);
			}
			else break;
		}
	}
}
```

Returns the hit position of a casted ray with origin `ro` and direction `rd` of an object defined in the `scene()` function. `p` returns the `vec3` hit position with `EPS` denoting the resolution of declaring a hit. A global `float refd` is used to determine if a reflecting object was hit and the ray has to be reflected. The value of `refd` is updated in each call of the `scene()` function.

## Camera setup

```GLSL
mat3 camera(vec3 ro, vec3 t, float r) {
	vec3 f = normalize(t-ro);
	vec3 u = vec3(sin(r),cos(r),0.);
	vec3 s = normalize(cross(f,u));
    return mat3(s, u, -f);
}
```

Returns a `mat3` to be multiplied with the ray direction `ro` (typically `vec3 rd = normalized(vec3(uv.xy, -1.));`) to get the proper ray direction for raymarching. The given `vec3 t` defines the lookat target and `float r` the rotation angle of the up vector. Formular from [khronos.org](https://www.khronos.org/registry/OpenGL-Refpages/gl2.1/xhtml/gluLookAt.xml).


# Normal calculation

```GLSL
vec3 normal(vec3 p) {
	vec2 e = vec2(EPS, 0.);
	float d = scene(p);
	return normalize(vec3(scene(p+e.xyy)-d, scene(p+e.yxy)-d, scene(p+e.yyx)-d));
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
