<!--
author:   Niklas

email:    your@mail.org

version:  0.0.1

language: en

narrator: US English Female

comment:  Try to write a short comment about
          your course, multiline is also okay.

link:     https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.css

script:   https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.js

-->

# Fast Inverse Square Root 

``` cpp
float Q_rsqrt(float number)
{
  long i;
  float x2, y;
  const float threehalfs = 1.5F;

  x2 = number * 0.5F;
  y  = number;
  i  = * ( long * ) &y;                       // evil floating point bit level hacking
  i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
  y  = * ( float * ) &i;
  y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
  // y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

  return y;
}
```

# Motivation
![](https://preview.redd.it/zzmfvxaedks51.png?width=1080&crop=smart&auto=webp&s=89cafddd31ead94c98d004a9a8decb4660cf53c8)

In 1993, id Software's "DOOM" revolutionized gaming by bringing a 2.5-dimensional world to the screens of millions.

![](https://images.igdb.com/igdb/image/upload/t_720p/tu5xtknz9ilfilspxzlp.jpg)

A year later, EA released "Magic Carpet", which implemented a real 3D environment, but with rather flat lighting.

Extensive lighting systems need surface normals to compute angles of incidence and refelction. Floating-point division were pretty difficult for the hardware of the time, so an efficient algorithm needed to avoid them as much as possible.

# How it works

``` cpp
float Q_rsqrt(float number)
{
  long i;
  float x2, y;
  const float threehalfs = 1.5F;

  x2 = number * 0.5F;
  y  = number;
  i  = * ( long * ) &y;                       // evil floating point bit level hacking
  i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
  y  = * ( float * ) &i;
  y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
  // y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

  return y;
}
```

## Binary logarithm

``` cpp
  y  = number;
  i  = * ( long * ) &y;                       // evil floating point bit level hacking
```
These two lines copy the only parameter to a new location on the stack and then treat those bits as an integer.

## "what the fuck?"
``` cpp
  i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
```
This line does the actual calculation of $$y=\frac{1}{\sqrt x}
$$ by using the identity of $$
\log_2 (y) = -\frac{1}{2} \log_2 (x)$$.

By substituting the formula for the approximate logarithm we get $$
{\displaystyle {\frac {I_{y}}{L}}-(B-\sigma )\approx -{\frac {1}{2}}\left({\frac {I_{x}}{L}}-(B-\sigma )\right)}$$, where I is the bit pattern treated as an integer, L is the size of the mantissa and B is the exponent bias. Sigma is a tuning parameter.

This can be rewritten as $${\displaystyle I_{y}\approx {\tfrac {3}{2}}L(B-\sigma )-{\tfrac {1}{2}}I_{x}}
$$, so the "magic number" is just the result of packing all those constants together, and the bit shift is nothing more than a division by two.

## Base2 Exponential
``` cpp
  y  = * ( float * ) &i;
```
This turns the integer back to a float.

## Newtons Method
``` cpp
  y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
  // y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed
```
The whole calculation can also be written as ${\displaystyle f(y)={\frac {1}{y^{2}}}-x=0}$. Following Newton's Method for finding the zero of a function, a better approximation $y_{n+1}$ for $y_n$ can be calculated with $${y_{n+1} = y_{n}-{\frac {f(y_{n})}{f'(y_{n})}}}$$.

With ${\displaystyle f(y)={\frac {1}{y^{2}}}-x}$ and ${\displaystyle f'(y)=-{\frac {2}{y^{3}}}}$, we get $$
{\displaystyle y_{n+1}=y_{n}\left({\frac {3}{2}}-{\frac {x}{2}}y_{n}^{2}\right)={\frac {y_{n}\left(3-xy_{n}^{2}\right)}{2}}.}$$

# History

This particular implementation is from 1999's "Quake III", but the roots go back way further.

An unpublished paper from 1986 first describes how to use bit hacking and Newton iterations to calculate a square root.
Greg Walsh at Ardent Computers most likely invented the algorithm in its full form in the early 90's, and a 1997 article in the IEEE Computer Graphics and Applications magazine showed a first simple approximation.

Intel released the Streaming SIMD Extensions to their x86 architecture in 1999, which made the code pretty much obsolete because it implemented square root operations in Assembly.

The internet of the early 2000s leaked and spread the code snippet.

# Sources
https://en.wikipedia.org/wiki/Fast_inverse_square_root
https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions
https://www.reddit.com/r/gaming/comments/26kjm8/comment/chs01my/
https://www.netlib.org/fdlibm/e_sqrt.c