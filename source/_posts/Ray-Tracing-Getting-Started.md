---
title: 'Ray Tracing: Getting Started'
date: 2020-09-13 21:05:36
tags: 
- Ray Tracing
mathjax: true
---

![First Final Scene](/images/FirstFinalScene.png)

## Overview

From [pbrt](http://www.pbr-book.org/3ed-2018/Introduction/Photorealistic_Rendering_and_the_Ray-Tracing_Algorithm.html) I quote:

>Although there are many ways to write a ray tracer, all such systems simulate at least the following objects and phenomena:
>- *Cameras:* ...
>- *Ray–object intersections:* ...
>- *Light sources:* ...
>- *Visibility:* ...
>- *Surface scattering:* ...
>- *Indirect light transport:* ...
>- *Ray propagation:* ...

As shown in the diagram below, from the camera we shoot a **view ray** towards every pixel in image space and track how they interact with objects/lights in the scene. And finally we have a color value for every single pixel. 

![ray trace overview](/images/RayTraceDiagram.svg)

Personally I learned rasterization before learning ray tracing, internally they have opposite philosophy: from the world to your eye or how your eye sees the world. And compare to all the approximation and interpolation method we use in **rasterization**, **ray tracing** is a more intuitive, realistic and yet resource consuming way for image rendering. 



## Ray 

Since it's called ray-tracing, obviously we need to represent a ray. A ray is denoted by formula
$$
\mathbf{P}(t) = \mathbf{origin} + t * \mathbf{direction}
$$
Accordingly we have a `Ray` class

```C++
class Ray
{
public:
	// ctor/dtor
	Ray() = default;
	~Ray() = default;
	Ray(const Vector3& org, const Vector3& dir, double time = 0.0)
		: m_origin(org), m_direction(dir), m_time(time)
	{
		m_direction.normalize();
	}

	// Getter
	Vector3 getOrigin() const { return m_origin; }
	Vector3 getDirection() const { return m_direction; }
	double getTime() const { return m_time; }

	// p(t) = origin + t * dir;
	Vector3 pointAt(const double& t) const { return m_origin + m_direction * t; }

private:
	Vector3 m_origin;
	Vector3 m_direction;
	double m_time;
};
```

## Camera

Contrary to rasterization, most of the operations we have is in world space, thus we need to transform screen space pixel coordinates into world space and shoot a view ray accordingly. 

$u, w, v$ represents axes of camera space, which is also right-hand. Then `vfov` and `aspect_ratio`  determines the view frustum. Then we have a `Camera` class:

```c++
class Camera
{
public:
	Camera(
		Point3  lookfrom,
		Point3  lookat,
		Vector3 vup,
		double  vfov,  // vertical field-of-view in degrees
		double  aspect_ratio,
		double  aperture,
		double  focus_dist,
		double t0 = 0, 
		double t1 = 0
	)
	{
		auto theta = degreeToRadian(vfov);
		auto h = tan(theta / 2.0);
		auto viewport_height = 2.0 * h;
		auto viewport_width = aspect_ratio * viewport_height;

		w = (lookfrom - lookat).getNormalied();
		u = vup.crossProduct(w).getNormalied();
		v = w.crossProduct(u).getNormalied();

		m_origin = lookfrom;
		m_horizontal = focus_dist * viewport_width * u;
		m_vertical = focus_dist * viewport_height * v;
		m_lowerLeftCorner = m_origin - m_horizontal / 2.0 - m_vertical / 2.0 - focus_dist * w;

		m_lens_radius = aperture / 2;
		m_time0 = t0;
		m_time1 = t1;
	}

	Ray getRay(double s, double t) const
	{
		Vector3 rd = m_lens_radius * Vector3::randomInUnitDisk();
		Vector3 offset = u * rd.x + v * rd.y;
		return Ray(
			m_origin + offset,
			m_lowerLeftCorner + s * m_horizontal + t * m_vertical - m_origin - offset,
			random_double(m_time0, m_time1)
		);
	}

private:
	Point3 m_origin;
	Point3 m_lowerLeftCorner;
	Vector3 m_horizontal;
	Vector3 m_vertical;
	Vector3 u, v, w;
	double m_lens_radius;
	double m_time0, m_time1; // shutter open/close time
};
```



## Ray-Object Intersections

### Base class

We create a `HitRecord` struct to keep record of the hit information we need. And a virtual base class for other objects that can be hit.

```c++
class Material;

struct HitRecord
{
	Point3 position;
	Vector3 normal;
	shared_ptr<Material> mat_ptr;
	double t;
};


class Hittable
{
public:
	Hittable() = default;
	virtual ~Hittable() = default;
	virtual bool hit(const Ray& r, const double& t_min, const double& t_max, HitRecord& rec) const = 0;
};
```

### Sphere 

For now we only have one primitive geometry - sphere, which make the  intersection math very simple, we know a sphere is denoted by formula:
$$
(x-\mathbf{c}.x)^2 + (y-\mathbf{c}.y)^2 + (z-\mathbf{c}.z)^2 = R^2
$$
And we put $\mathbf{P} = \mathbf{P}(t) = \mathbf{O} + t\mathbf{D}$ into the equation:
$$
(\mathbf{D} \cdot \mathbf{D})t^2 + 2(\mathbf{D} \cdot (\mathbf{S} - \mathbf{c}))t + (\mathbf{S} - \mathbf{c}) \cdot (\mathbf{S} - \mathbf{c}) - R^2 = 0
$$
For $at^2 + bt + c = 0$ we have
$$
t = \frac{-b\pm \sqrt{b^2-4ac}}{2a}
$$

where $b^2−4ac$ is the **discriminant** of the equation, and

- If $discriminant<0$, the line of the ray does not intersect the sphere (missed);
- If $discriminant=0$, the line of the ray just touches the sphere in one point (tangent);
- If $discriminant>0$, the line of the ray touches the sphere in two points (intersected).

Then we have the `Sphere` class


```c++
class Sphere : public Hittable
{
public:
	Sphere(){}
	Sphere(Point3 center, double r, shared_ptr<Material> m) : m_center(center), m_radius(r), m_mat_ptr(m) {}

	virtual bool hit(const Ray& r, const double& tmin, const double& tmax, HitRecord& rec) const override;

public:
	Point3 m_center;
	double m_radius;
	shared_ptr<Material> m_mat_ptr;
};

bool Sphere::hit(const Ray &r, const double &tmin, const double &tmax, HitRecord &rec) const
{
	Vector3 oc = r.getOrigin() - m_center;
	double a = r.getDirection().getSquaredLength();
	double half_b = oc.dotProduct(r.getDirection());
	double c = oc.getSquaredLength() - m_radius * m_radius;
	double discriminant = half_b * half_b - a * c;
	if (discriminant > 0.0)
	{
		double tmp = (-half_b - sqrt(discriminant)) / a;
		if (tmp > tmin && tmp < tmax)
		{
			rec.t = tmp;
			rec.position = r.pointAt(rec.t);
			Vector3 outward_normal = (rec.position - m_center) / m_radius;
			rec.setFaceNormal(r, outward_normal);
			Sphere::getSphereUV((rec.position - m_center) / m_radius, rec.u, rec.v);
			rec.mat_ptr = m_mat_ptr;
			return true;
		}

		tmp = (-half_b + sqrt(discriminant)) / a;
		if (tmp > tmin && tmp < tmax)
		{
			rec.t = tmp;
			rec.position = r.pointAt(rec.t);
			Vector3 outward_normal = (rec.position - m_center) / m_radius;
			rec.setFaceNormal(r, outward_normal);
			Sphere::getSphereUV((rec.position - m_center) / m_radius, rec.u, rec.v);
			rec.mat_ptr = m_mat_ptr;
			return true;
		}
	}
	return false;
}
```

### List of hittable objects

For multiple objects in the scene we can have a list of these objects as it's one giant object.

```C++
class HittableList : public Hittable
{
public:
	HittableList() = default;
	HittableList(shared_ptr<Hittable> object) { add(object); }
	
	void clear() { m_list.clear(); }
	void add(std::shared_ptr<Hittable> object) { m_list.push_back(object); }
	bool isEmpty() const { return m_list.empty(); }

	virtual bool hit(const Ray& r, const double& tmin, const double& tmax, HitRecord& rec) const override;

public:
	std::vector<std::shared_ptr<Hittable>> m_list;
};

bool HittableList::hit(const Ray &r, const double &tmin, const double &tmax, HitRecord &rec) const
{
	HitRecord temp_rec;
	bool hit_anything = false;
	double closest_so_far = tmax;

	for (int i = 0; i < m_list.size(); i++)
	{
		if (m_list[i]->hit(r, tmin, closest_so_far, temp_rec))
		{
			hit_anything = true;
			closest_so_far = temp_rec.t;
			rec = temp_rec;
		}
	}

	return hit_anything;
}
```

## Material

A material basically describes what an object will do to the ray when the ray hits the object - how it reflects, refracts and so on. We design a interface `scatter` to describe such behavior.

```C++
class Material
{
public:
	virtual bool scatter(const Ray& r_in, const HitRecord& rec, Color& attenuation, Ray& scattered) const = 0;
};
```

### Lambertian



### Metal



### Dielectric



## Antialiasing

Without antialiasing the image tend to have noisy and alias, we simply do a uniform-random sample in a square without considering the sample PDF(probability density function) :

```C++
int samples_per_pixel = 100;
for (int s = 0; s < samples_per_pixel; ++s)
{
	auto u = (i + random_double()) / (image_width - 1);
	auto v = (j + random_double()) / (image_height - 1);
	Ray r = cam.getRay(u, v);
	pixel_color += ray_color(r, background, world, max_depth);
}
write_color(buffer, i, j, image_width, pixel_color, samples_per_pixel);
```





## Depth of Field (Defocus Blur)