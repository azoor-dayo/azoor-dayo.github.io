	# General
For GLSL/GLM, use **COLUMN MAJOR** format.
```
0 3 6
1 4 7
2 5 8
```

Different of the usual ROW MAJOR format.
```
0 1 2
3 4 5
6 7 8
```

Terms: **Canonical** means origin at bottom left

3D Pipeline: `Model Coords > World Coords > View Coords(Cam local system) > Lighting stuff > Project onto Camera plane > Clip Coords > Persepctive division > NDC *OPENGL TAKES OVER* > Screen/Device Coords > Framebuffer` 

2D Pipeline: `Model Coords > World Coords > View Coords(Cam local system) > NDC *OPENGL TAKES OVER* > Screen/Device Coords > Framebuffer`

NDC: Origin at center, -1 to 1.
Viewport: Origin Bottom Left, 2x2.

Screen/Device Coords System: 
Origin TOP LEFT, size is dependent. If full device resolution, 1920x1080. If a window, then whatever the window size is e.g. 800x600. Windows OS has this layout.

> OpenGL does conversion from NDC to Screen/Device Coords automagically.
# Line Rasterization
Using y=mx+c, we can calculate which pixel is on the line as we iterate through each pixel on X axis. But we don't wanna run the m\*x + c every iteration, more expensive. Instead, find the increment of y for every x (aka the m value) and use that to apply iteratively. 

`next x = last x + m`. 

Pros: Avoids a mult operation 
Cons: Will lose accuracy if done for long periods due to float inaccuracy. Does calculations in floats. Not the best.
![[Pasted image 20240819004713.png]]

If slope is > 1, iterating across x will miss some pixels cos the behind pixel is being "covered". Instead, iterate via y instead, and use the slope of run/rise instead.
![[Pasted image 20240819004730.png]]
![[Pasted image 20240819004735.png]]
Step 1: Iterate thru X values or Y values, depending on the slope and what startpoint/endpoint order is. Prepare a dy/dx or dx/dy value to increment to the not-iterating value. e.g. if iterating x, each x++ is +=dy/dx for y. 

Step 2: Let's say iterate Y value. Take the BOTTOM LEFT of each cell and see what X value it is. 

Step 3: Cell to be colored will be the Y value and X value! Visually, it's going to be the BOTTOM LEFT edges of the cell that is being "marked" by those 2 values. Step 4: iterate all the way, done. Make sure to leave the last cell unset if need line loops or sth.

![[Pasted image 20240819004819.png]]

Full Video Explanation:
Bresenham's Algorithm
https://youtu.be/RGB-wlatStc?t=885

Idea: For each X iteration, a Y value will be provided from the PREVIOUS iteration (unless it's the first iteration, Y value is alr given as the first point). That Y value will be either the same as the last Y value, or Y++. This algorithm figures out if the NEXT Y value should be ++ or not. 

To figure out if ++ or not, we use if( (d1-d2) > 0). If this d1-d2 is -ve, use BOTTOM PIXEL. Else, TOP PIXEL.
![[Pasted image 20240819004904.png]]
d1 is calculated by the actual Y value of the line - current Y value d2 vice versa. 

`d1 - d2` is BOTTOM minus TOP. If value is +ve, bottom is fatter, hence line closer to TOP. Else closer to bottom. 

Take note if going the opposite way on the x or y axis will result in a different formula! 

But hol up, we still need to use m! That's a float and das not good.
![[Pasted image 20240819004933.png]]
we sub m = dy/dx and do algebra on it, then apply dx on both sides. now we got `dx(d1 - d2) = 2dy*xk - 2dx*yk + 2dy+2dx*c-dx` 
This is a decision parameter, let's call it Pk. Prata calls it dk.
![[Pasted image 20240819004954.png]]
In formula `P(k) = 2dy*xk - 2dx*yk + 2dy+2dx*c-dx`, the last half `2dy+2dx*c-dx` got removed cos the next few steps are to find `P(k + 1) - P(k)`, and the constants would have cancelled each other out instantly.

`P(k + 1) - P(k)` is useful for doing _incremental_ calculations for the CPU for optimization. However, as a human, using P(k) is good enough.

If P(k) is +ve, TOP pixel! Else, BOTTOM pixel! **You probably cannot memorise this formula as it differs if you are doing in the negative direction.**

However, when actually drawing lines, who the fuck cares, make your line start bottom left and go top right.

Oh, and the c in the formula comes from y=mx+c.

Do this: 
Using `y= mx + c`, sub in `dy/dx` and then rearrange to get `2dx*c = ???` At the end of the day you should get the equation for P(k) = xxx that just needs: the current (k) X and Y value, dy and dx.
![[Pasted image 20240819005048.png]]
and also one smol one for finding initial P
![[Pasted image 20240819005056.png]]
OK GRAPE TLDR FOR BRESENHAM:
These formulas ONLY WORK for **positive iterations**. If you need to iterate backwards, well just don't lol. You can always choose to iterate forwards. Backwards will require a different formula that is really only useful for prata's exams.
Check if each dy/dx value is +ve or -ve to determine the octant, and use that to decide each pixel should += 1 or -= 1 in X and Y directions.
**MODULUS THE dy AND dx VALUES FIRST BEFORE CONTINUING.** This is making sure you're iterating forwards ON THE X AXIS, as the formula only works this way.
```
Pinitial = 2dy - dx  // P1 to start out with
P+ve = 2dy - 2dx     // If current P value is +ve, add this value to get next P.
P-ve = 2dy           // If current P value is -ve, add this value to get next P.
```
If P is +ve this iteration, next step's y will += 1. If -ve, REMAIN AS SAME VALUE.

OK grape, this is for slopes 0-1 for both +ve and -ve. How about 1-inf? Just flip the dys with the dxs in the formula, GRAPE. Also, be sure to iterate along the Y axis this time, not X.

## Incremental Line Rasterization
Incremental edge equation: ax+by+c = eval 
1. Calculate eval at desired starting x,y point (bottom left). 
2. As you iterate over x++, the new (x+1, y) eval will be eval + a. Delta in x dir is a. 3
3. As you move up to the next row using y++, take the eval value RIGHT BELOW the new point (not the one at the right side after u x++'d) and + b. so (x, y+1) will be eval + b. Delta in y dir is b.
# Triangle Rasterization
When you want to "color" a grid of squares to best represent a triangle on the screen.

## Edge Walking
An easy method, but inefficient.
**Edge Walking method:** If triangle has no horizontal edge, use the MIDDLE vertex to draw a horizontal line to break a triangle into 2 pieces. Also, take note if this point is to the LEFT or RIGHT. For the 1/2 pieces of triangle, use the 2 corresponding line equations to compute the SPAN. For y = 5, what are the 2 x values for each line? Then iterate through these X values to calculate the fragment positions. 

PROS: Easy to use after implementation 
CONS: Too many edge cases. Complicated setup. Middle point to left or right? Do i need to cut the triangle? Cannot be parallelised. Serial computation.
![[Pasted image 20240819013535.png]]

## Edge Equation
A more sophisticated method: **Edge Equation Method:**
For each fragment, calculate if this current fragment is within the 3 lines. Each line will divide 2D plane into 2 regions, "inside" or "outside". If fragment is "inside" all 3 line equations, then it is inside the triangle.

So it's just 3 dot product checks per fragment. Neat!
![[Pasted image 20240819013655.png]]

For standardization, use the LEFT normal. When dot product'ing a fragment (vector) to the normal, +ve means it's to the left of the line, which is "inside". This conforms with ccw winding of triangles, so DON'T use the right side normal. Given the vector, the left normal is: `[-v.y, v.x]`. Normalize as needed.
![[Pasted image 20240819013713.png]]

So now, the problem will be getting the line equation to do the computation of this inside-outside thing, from the 3 points of the triangle.

For a given line, create an edge equation that returns a bool to see if fragment is inside or outside. Using point-normal formula explicit format: `L : ax + by + c == 0`, derived from `L: n . (x - p) == 0` Need to find `a`, `b` and `c`. `a` and `b` are the x and y values of the normal, `c` is just some fucked up value.
![[Pasted image 20240819014108.png]]
Hence the function should be: Take in point p and q, and variable point x. If (ax + by + c) >= 0 then TRUE, else FALSE.

Now here comes another issue: If the point/fragment/pixel lies directly on the the imaginary line, is it in or out? In other words, if (ax + by + c) == 0 then what do?

Solution: Top left rule. If this line is a "top" line or "left" line of the triangle, then consider it inside. Else, out.
```
Pre-computing Is Top or left edge boolean
if (a != 0) {
  if (a>0) return true; // left edge
  else return false; // right edge
} 
else (a == 0){
  if (b<0) return true; // top
  else return false; // not top
}
```
![[Pasted image 20240819014201.png]]

How to tell if a line/edge is a top/left/right?

Identifying Top Edges and Left edges: 
Is Top when: If line is HORIZONTAL (X value of normal is == 0) and both y-values ABOVE the point/fragment/pixel.
![[Pasted image 20240819015000.png]]
Is Left when: X value of normal is > 0. V is always pointing DOWNWARDS. Based on CCW winding, the next point that allows for CCW is always to the RIGHT. 
In `ax + by + c == 0`, `a == 0` and y values above last point is HORIZONTAL EDGE. `a > 0` is LEFT EDGE
![[Pasted image 20240819015014.png]]
Note: If the top edge isn't perfectly HORIZONTAL, then it's either a left or right edge. There can also be 2 left edges. 

Also, when calculating if a PIXEL is in or out of the triangle, use its MID POINT. So the int coords of a fragment (x,y) will need to become (x + 0.5, y + 0.5) 

Calculate ax + by + c for every line. If value is +ve for all, fragment is INSIDE triangle.

Therefore, TLDR:
```
Pre-computing Is Top or left edge boolean for each edge:
if (a != 0) {
  if (a>0) return true; // left edge
  else return false; // right edge
} 
else (a == 0){
  if (b<0) return true; // top
  else return false; // not top
}
```
# Texture Mapping Basics
## Texture Space VS Texel space
TLDR: Texture space is NORMALIZED while Texel is in RAW PIXELS, INT ONLY. Note: Texel space can be used to represent a view port, OR the raw image itself (since the size of a raw image is pixels x pixels)
![[Pasted image 20240819005324.png]]
![[Pasted image 20240819005329.png]]
With mapping, different methods of applying texture becomes available, such as:
Tiling, stretching, mirroring etc.

# Texture Blending (Decal)
https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBlendFunc.xhtml
Consider an object with existing color. You apply a texture onto the object, and you want it to kind of "fade" in and out of the underlying texture, like a disappearing blood stain fading away.

One method is to interpolate between these 2 textures using a singular value to determine if one texture or another is fully shown, or in-between.

The formula is:
`Final Color = InterpAmount * TexColor + (1 - InterpAmount) * ObjColor`

InterpAmount is the interpolation amount, and should be between 0 to 1.
When value is 1, only TexColor is shown.
When value is 0, only ObjColor is shown.
Anything in between, there will be a color blend!
# Back-face Culling
Cross Products detect if a set of points are backfacing or not **if p0, p1 and p2 are in ccw order, then (p1-p0) cross (p2-p0) SHOULD be +ve.**
![[Pasted image 20240819015232.png]]

# Left/Right hand coordinate systems
v1 cross v2 using right thumb rule. If match, is right hand. If not match, left hand.
![[Pasted image 20240819015340.png]]

# Interpolation
## Linear
Simple shit.
```
float Lerp (float start, float end, float t) {
  return (end - start) * t + start
}
```
## Bilinear
Bilinear Interpolation. Used when finding a middle pt between 4 2D points (in a parallelogram) using 2 ratios of the 2 edges.
```
float BilinearInterp (Vec2 p0, Vec2 p1, Vec2 p2, Vec2 p3, float s, float t) {
  Vec2 p01 = Lerp(p0, p1, s);
  Vec2 p32 = Lerp(p3, p2, s);
  return Lerp(p01, p32, t);
}
```
![[Pasted image 20240819015600.png]]
## Barycentric
Used to find a middle point of a triangle with 2 or 3 ratio values. If 2 of them are s and t, 3rd one will always be (1 - s - t). All weights need to add up to 1. So middle of triangle will have each weight be 0.33.

### Point-Vector interpretation of Barycentric Coordinates
TLDR: (1-t1-t2) is always paired with the original point (p0). 
t1 and t2 are paired with their "destinations" extending out from p0. 
`(1-t1-t2)p0 + t1p1 + t2p2 = a point inside triangle.`

Used when: 
GIVEN 3 PTS 
GIVEN 2/3 RATIOS 
NEED TO FIND POINT
![[Pasted image 20240819015705.png]]

### Area-Of-Triangle interpretation of Barycentric Combinations
Uses the area of each triangle segment over the entire triangle area to get a ratio. This ratio is the weight for each point. Ratio will correspond the the barycentric coordinates of t0 t1 and t2.
Kinda works on the principle that ratio of areas will all normalize to sum up to 1.

Used when: 
GIVEN 3 PTS 
HAVE MID POINT 
NEED TO FIND RATIO

TLDR: Ratio of A0 is paired with p0 (opp side), etcetc. 
The t0/t1/t2 that triangle segment ratios give will be paired to the OPPOSITE point.
![[Pasted image 20240819015824.png]]
Calculation:
For 3D (and how 2D was derived too):
`Area of triangle = 0.5 * base * height`.
Let base be `p1-p0`. Height isn't aligned with axis, so use projection.
Project either SLANTS of triangle onto normal of base.
`Let normal of base be Nn normalized normal be n`
`2 possible normals, only take the normal in same dir as P2`
Normal of base, normalized: `N / ||N||`
Slant of triangle: `p2-p1`
Height = slant projected onto base normal. `height = (p2-p1).(N/||N||)`
Slap everything together. `Area of triangle = 0.5 * ||p1-p0|| * (p2-p1).(N / ||N||)`
Oh look u can cancel out `||p1-p0||` with `||N||`, because the normal of the base has the same magnitude.
`Area of triangle = 0.5 * (p2-p1).Nc`
`2 * Area of triangle = (p2-p1).N` For optimization, since we'll divide both values with a 2x later anyway.
Again, for optimization, pre-calculate value of 1/total area first then use that to multiply every loop.

If you convert points into 2D, you will get the **cross product formula for z component**! 
How handy.

For 2D: Use the Z value (magnitude) of the cross product of p1-p0 cross p2-p0 to get **2\*area of triangle**. (u1v2 - u2v1)

Then, 2\*segment area / 2\*total area = ratio. Also, ratio of last triangle segment will just be 1-t1-t2, no need to calculate the area.
#### Incremental Barycentric
Incremental barycentric equation: (1 - t1 - t2)p0 + t1p1 + t2p2 = new p
Use triangle-area based eqn instead: (A0/A)p0 + (A1/A)p0 + (A2/A)p2 = new p 
1. Calculate new p at desired starting x,y point (bottom left). 
2. 2. Calculate deltas. Delta in x dir is ???.
# Vertex Pipeline
The geometry/vertex pipeline is as follows:
`Model > World > Camera > Lighting (if any) > Project (onto 2D cam Plane) > Clip > Persepctive division > NDC > Screen/Device Coords > Framebuffer` 
![](https://upload.wikimedia.org/wikipedia/commons/f/f8/Geometry_pipeline_en.svg)
## Model
	The raw vertex points straight from a model file (.obj, .fbx etc).
These points work off 0,0,0 as the center of the model.
## World
AKA the **Model Transform**. Model > World. Exists for each unique object in the world.
Applies the model's position, scale and rotation to "place" it into the world, with data from the Transform component.
Concatenate the Scale, Rotation and Transform matrices together to get this matrix (in the order TRS).
## View (Camera)
AKA the **View Transform**. World > Camera. Exists for each unique camera.
Brings the objects in the world to the camera's local coordinate system.

We just want to rotate all objects in the world and shift them relative to where the camera is in world space, so that the camera is now at 0,0,0. Simple, just find the e1' e2' e3' vectors. No need to calculate rotation.

Data required: Camera position, look direction, global up direction.

1. Determine if your camera should use a LEFT-HANDED or RIGHT-HANDED coordinate system.
   OpenGL and friends use a RIGHT-HANDED system, while DirectX uses LEFT-HANDED.
   Visualisation: [https://www.youtube.com/watch?v=TGbMzoJqV7c](https://www.youtube.com/watch?v=TGbMzoJqV7c "https://www.youtube.com/watch?v=TGbMzoJqV7c")
   ![[Pasted image 20240819232059.png]]
   The difference between a left/right hand system is if the positive Z-axis comes at your camera or goes away from the camera. The X and Y axis directions should remain the same for both systems.
2. Figure out e3' first (Z/blue axis). For RIGHT-HANDED system, take camPos - targetPos (vector towards camera). For LEFT-HANDED system, the other way (vector away from camera). Normalize this vector, you got e3'.
3. Next, figure out e1'. e2' (the camera y axis), e3' and global up and  all lie on the same plane. This means you can cross e3' and the global up to find e1' (X/red axis). Assuming global up is a unit vector, this cross product should result in a unit vector for e1', no normalization needed.
   >Note: Be aware of the order of cross product! For LEFT-HANDED system, do e3' X globalUp. For RIGHT-HANDED system, do globalUP X e3'.
4. Lastly, you can find e2' by crossing e1' and e3' together. Again, be wary about the cross product order.
   > For LEFT-HANDED system, do e1' X e3'. For RIGHT-HANDED system, do e3' X e1'.

Finally, throw e1' e2' e3' into a matrix.
```
| e1'.x e2'.x e3'.x camPos.x |
| e1'.y e2'.y e3'.y camPos.y |
| e1'.z e2'.z e3'.z camPos.z |
|  0     0     0       1     |
```
But hold on, this matrix is for LOCAL to GLOBAL. **ALL default e1'e2'e3' matrices are LOCAL TO GLOBAL.** So, we need to inverse that matrix! Inverting 3x3 is a pain... but thank goodness it's easy in this case. There's a special property of orthonormal 3x3 matrices (all 3 columns are orthogonal and normalized): the inverse of this matrix is the same as the transpose of this matrix! So we transpose the above matrix (the 3x3 portion) to get the below:
```
| e1'.x  e1'.y  e1'.z  -e1'·camPos |
| e2'.x  e2'.y  e2'.z  -e2'·camPos |
| e3'.x  e3'.y  e3'.z  -e3'·camPos |
|   0      0      0         1      |

Note: 4th col is a dot product of 2 vectors, and not a regular period.
```
https://stackoverflow.com/questions/2624422/efficient-4x4-matrix-inverse-affine-transform
Potential way to speed up Ax=b operations without finding A inverse: https://www.johndcook.com/blog/2010/01/19/dont-invert-that-matrix/
## Lighting
From Wikipedia:
```
Often a scene contains light sources placed at different positions to make the lighting of the objects appear more realistic. In this case, a gain factor for the texture is calculated for each vertex based on the light sources and the material properties associated with the corresponding triangle. In the later rasterization step, the vertex values of a triangle are interpolated over its surface. A general lighting (ambient light) is applied to all surfaces. It is the diffuse and thus direction-independent brightness of the scene. The sun is a directed light source, which can be assumed to be infinitely far away. The illumination effected by the sun on a surface is determined by forming the scalar product of the directional vector from the sun and the normal vector of the surface. If the value is negative, the surface is facing the sun. 
```
## Projection
2 Types of projection: Orthographic and Perspective.

Orthographic Projection (aka Parallel projection): projection directions for all points are PARALLEL. Meaning the "clipping box" is a cuboid, not a trapezoid. 

Perspective projection: projection directions for all points are NOT PARALLEL. The "clipping box" trapezoid. All pixels from the start to end get scaled down accordingly to fit on the end plane. 'Gives the "fish eye" kind of view.
### Orthographic Projection
Re-maps points in a volume in view space and plonks them into a 2x2x2 NDC space. Your cuboid volume in view space must be aligned to the axes, and can't be in an arbitrary position or shape.
6 values to define the bounding volume in view space are specified to define this volume. (left, right, bottom, top, near, far)
![[Pasted image 20240825204651.png]]
A few things need to happen:
1. Transform (move) the volume to be centered at origin
2. Scale the points in the x, y, and z axes to fit into the 2x2x2 NDC space.
#### Transform
Find the center of your arbitrary volume using the 6 values, and then apply a transform. Simple.
```
| 1 0 0 -Px |
| 0 1 0 -Py |
| 0 0 1 -Pz |
| 0 0 0  1  |
```
#### Scale
Find the size of your bounding volume, normalize the points into a 1x1x1 box, then x2 to the values to fit into a 2x2x2 volume.
Working on a singular axis for visualization:
```
Let point be x
Let box width be w

Map x from a w*h*d box into a 2*2*2 box:
x' = x / w * 2

Matrix to apply to point:
| 2/w  0   0  0 |   | x |
|  0  2/h  0  0 |   | y |
|  0   0  2/d 0 | * | z |
|  0   0   0  1 |   | 1 |
```
Of course, replace Px etc and w/h/d with the 6 corner values if needed.

Concat the 2 matrices together, you get this:
```
| 2/w  0   0  -Px |   | x |
|  0  2/h  0  -Py |   | y |
|  0   0  2/d -Pz | * | z |
|  0   0   0   1  |   | 1 |
```
![[Pasted image 20240826220306.png]]
One last caveat. OpenGL does its Z axis in the negative direction, so flip the Z value in the matrix to ensure your Z value in the NDC remains positive.
### Simple Perspective Projection
[Youtube Explanation](https://www.youtube.com/watch?v=U0_ONQQ5ZNM)
Use the depth (Z) to calculate how much to map the X and Y values. Assume the Center of Projection is at origin.

Define your projection plane (aka near plane) using a Z value, for example, -5. Denoted as Nz. Have a point, let its Z value be -6. Denoted as Pz.
![[Pasted image 20240826220851.png]]
To project the point P onto the projection plane, Pz' should be equal to Nz (-5). However, the X and Y values should be "interpolated". The value to interpolate by is the ratio of Nz to Pz.

To find `Px'` and `Py'`, do: 
`Px' = Px / Pz * Nz`

Apply this formula to `Px, Py` and `Pz` to get `Px', Py'` and `Pz'`. 

Rewriting this to become easier to work with will give: 
`Px' = Px * Nz / Pz`

`Pz'` SHOULD end up equal to `Nz`.

>Note: If the sign of both `Pz` and `Nz` values are different, it means the projection plane is on the wrong side, or your point is behind the camera. Either cull the point, or the point will be rendered upside down and the player can see behind the camera.

To express this in a matrix WITH perspective division, write it as such:
```
| Px * Nz |   | Nz  0  0  0 |   | Px |
| Py * Nz |   |  0 Nz  0  0 |   | Py |
| Pz * Nz | = |  0  0 Nz  0 | * | Pz |
|   Pz    |   |  0  0  1  0 |   | 1  |
```
With the resultant matrix, divide the x y and z values with the w whenever convenient to get the final `Px' Py' Pz'` values.

Again, note that both your `Pz` and `Nz` values are negative if the camera is pointing towards the -Z axis. Having the final divisor w be a negative will fix the x, y being negative and make z negative. Replacing the "1" with a "-1" in the matrix will do just that.

From here, you can map the values to NDC by slapping it through a Screen Space to NDC matrix or something (now working in 2D space), or try to expand on this idea more by doing a full perspective projection.
### Full Perspective Projection
Now, we want to "preserve" the `Pz'` value after our projection.
Reviewing what we have previously:
```
| Nz  0  0  0 |   | Px |   | Px * Nz |   | Px * Nz / Pz |
|  0 Nz  0  0 |   | Py |   | Py * Nz |   | Px * Nz / Pz |
|  0  0 Nz  0 | * | Pz | = | Pz * Nz | = | Pz * Nz / Pz |
|  0  0  1  0 |   | 1  |   |   Pz    |
```
Focusing on the final Z value: `Pz * Nz / Pz`
We can see that the z value of the evaluated point always resolves to Nz at the end. That's not good.

We'd want our final Z value to match the original `Pz` value of the original point in the ideal scenario. This means that we can mostly ignore `Nz` for now and try to obtain `Pz` as our final Z value.
This means that, for the previous step, instead of obtaining `Pz * Nz`, we should ideally obtain `Pz`^2
```
| Nz  0  0  0 |   | Px |   | Px * Nz |   | Px * Nz / Pz |
|  0 Nz  0  0 |   | Py |   | Py * Nz |   | Px * Nz / Pz |
|  ?  ?  ?  ? | * | Pz | = |   Pz^2  | = | Pz * Pz / Pz |
|  0  0  1  0 |   | 1  |   |   Pz    |
```
The only 2 values in the matrix we have to work with are the Nz value and the 0 to its right. Let's label them m1 and m2.
```
| Nz  0  0  0  |   | Px |
|  0 Nz  0  0  |   | Py |
|  0  0 m1  m2 | * | Pz | = |   Pz^2  |
|  0  0  1  0  |   | 1  |
```
Focusing on the Z row, we can try to make an equation to obtain Pz^2 from this:
`m1*Pz + m2 = Pz^2`

Unfortunately, there isn't a straightforward solution. There is no way to obtain `Pz`^2 for all values of `Pz`.
This is now a quadratic equation, which results in a curve.

Solve for m1 and m2 simultaneously. This means we have to plug something into Pz. What values should we plug? Well, the near and far planes! (because what other values asre there to use lol)
When `Pz == n`, we get `m1*n + m2 = n^2`
When `Pz == f`, we get `m1*f + m2 = n^f`
Solve simultaneously to get: 
`m1 = f + n`
`m2 = -fn`

Plug this into the matrix:
```
| Nz  0  0   0  |   | Px |
|  0 Nz  0   0  |   | Py |
|  0  0 f+n -fn | * | Pz | = | Pz^2... sometimes |
|  0  0  1   0  |   | 1  |
```
This solution is not the most ideal, but it's the closest we can get to the ideal value Pz.
If we dig deeper, we can see that this final Z value has the property of `1/Pz`:
```
Focusing on Z row: [...matrix...] = f+n-fn(1/Pz) = final Z value
```
Problems arise when n and f are too far apart, and the values obtained are too close together, thus resulting in z fighting.
![](https://developer-blogs.nvidia.com/wp-content/uploads/2015/07/depth-perception-graph1-b.jpg)
More on Z-fighting due to precision loss: https://developer.nvidia.com/blog/visualizing-depth-precision/
To summarize:
- The closer your z values are to 0, the more bunched up these ticks on the Z-axis will get.
- Moving objects a tiny bit in this z range will cause your d (depth) value to jump much more than if you moved the same tiny bit further out towards the far plane, causing higher chance of Z-fighting
- Floating-point precision inconsistencies with slightly different camera angles can be enough to cause the tiny value changes on the z-axis
- Moving your near plane closer to 0 will cause more intense Z-fighting as more values reside on the steeper portions of the graph. Moving the near plane further out will reduce Z-fighting.

One way to combat this issue, Reverse Z: https://tomhultonharrop.com/mathematics/graphics/2023/08/06/reverse-z.html
![](https://developer-blogs.nvidia.com/wp-content/uploads/2021/05/depth-precision-graph5-625x324.jpg)
We map the 0 and 1 the other way round instead, so that when Z-fighting happens, it happens further away from the camera. Now, moving your far plane further away will result in a higher chance of Z-fighting, leaving the closer objects less prone to Z-fighting.

Aite great! Full perspective projection done, put it together with mapping into NDC and that's it.
## Clipping: Liang-Barsky Algorithm
![](https://upload.wikimedia.org/wikipedia/commons/0/01/Cube_clipping.svg)

![[Pasted image 20240826221425.png]]
![[Pasted image 20240826221432.png]]

Uses the principle of P' = P + tV

General Idea: Get the t value of line entry (tStart), and t value of line exit (tEnd).
![[Pasted image 20240826221522.png]]
Calculate the intersection of this line with every one of the 4 planes. You should get 4 t values: tLeft, tTop, tRight, tBottom. NOTE: we're calculating the t value, not the point itself.
For tStart, you'll want to compare the t values that are in this manner: OUT to IN. (Not always tLeft and tBottom!)
Vice versa for tEnd, you want to compare the t values that are IN to OUT.

for tStart, you want the biggest t value (out of the 2, and t=0), tEnd the smallest t value(out of the 2 and t=1). That will be your intersection t values.

**Properties**
- t value at 0 means the start point. t value at 1 means the end point. Usually we have 2 points, not a P+tV parametric equation.
- t values obtained can be out of the 0-1 range. any value out of this range is on this line but not within the line segment. That's why we cap values using 0 and 1 in our min/max functions.

**Calculating t**
Since plane is usually a normal, we do parametric and implicit line intersection.
Follow the below formula. 
normal n is the first 2 values of ax+by+c=0, a and b.
Point P is the starting point.
Vector V is End point - Start point.

Plug all values in and calculate t.
![[Pasted image 20240826221617.png]]
![[Pasted image 20240826221624.png]]

**Codifying**
![[Pasted image 20240826221720.png]]
Use a for loop to go thru all planes and calculate t.
Each iteration, you have 2 points and a plane (with implicit equation ax+by+c=0).
For each plane, check if your 2 points are going from OUT to IN of the plane, or the other way round.
Normals are pointing OUT.
U can use pre-computed values of the 2 points by subbing their x and y into ax+by+c. Result is distance from plane. -ve means IN, +ve means OUT.

NOTE: At this point, check for horizontal/vertical edge cases. Horizontal lines can't intersect with the left and right planes, vice versa for horizontal lines.

So if point 1 is +ve and point 2 is -ve, this t value should be considered for tStart.
If point 1 is -ve and other one is +ve, this t value should be considered for tEnd. 
![[Pasted image 20240826221740.png]]
Now u can have a method to distinguish t's that are for tStart and tEnd respectively.
Compare each value with a saved tStart/tEnd value, and override each iteration if it is more/less. Refer to first pic above.

FINALLY you have the proper tFirst and tLast values.

If tFirst is after tLast, aka tFirst > tLast, it means there is NO intersection
### Frustum Checking
**Using Clip-frame point itself to do inside-outside frustum check**

Let's say you have a matrix to bring a value to clip frame. The matrix is encoded such that if the w value is used to do perspective division, the result will be in NDC.
toClipFrame is a matrix designed to bring points to NDC via matrix multiplication AND persp division. But, we stop BEFORE persp division to do checking if point is in frustum or not.

"Ok grape, I have a point in clip frame. what do?"
If the camera is all aligned to the view origin and shit, GRAPE! Those 4 values x y z w are all you need to check if value is in or out of plane.

Follow dis handy formula:
Values below are calculated using ax+by+cz+d = 0 formula. Also means the result value is the distance of point to this plane.
If value is +ve, point is out of plane, cos the normal is pointing OUTWARDS.

```
Left:   -x - w
Right:   x - w
Bottom: -y - w
Top:     y - w
Near:   -z - w
Far:     z - w
```
For more info, cos this stupid matrix has NDC encoded in it, we can assume that each plane has nice values to use.

```
Left:   [-1 0 0 -1]
Right:  [1 0 0 -1]
Bottom: [0 -1 0 -1]
Top:    [0 1 0 -1]
Near:   [0 0 -1 -1]
Far:    [0 0 1 -1]
```

These values supply the a,b,c,d values in the formula ax+by+cz+d = 0.
So, subbing these values as a,b,c,d, and grabbing your existing point to sub in its x,y,z points (and also the w somehow), we can simplify each one into the table above.

# Lighting calulations
## Shading
https://en.wikipedia.org/wiki/Shading
Shading is done by using a light source and a surface normal to determine the final color of what a pixel should be, based on the cosine of the angle of the normal and the light source.

At 0 degrees (cos0 = 1), the light intensity should be 1. 
At 90 degrees (cos90 = 0), the light intensity should be 0. 
At >90 degrees, clamp light intensity at 0 (because you can't have negative light).

Dot product of 2 unit vectors conveniently gives us the cosine value, so we can use the dot product.
![[Pasted image 20240904221918.png]]

### Faceted (Flat) Shading
Super-fast shader.
Basic idea is that all pixels on a triangle surface uses the same normal for lighting calculation. Since triangles have a flat surface, this means that all pixels in this triangle will have the same color value. The normal should be the triangle surface's normal.
![[Pasted image 20240904221948.png]]

End result will look very... triangle-y. Each triangle face will be visibly shaded, but it's super fast. 
![](https://help.autodesk.com/cloudhelp/2023/ENU/MotionBuilder/images/GUID-A9B5C225-B9B6-4959-AB74-E5BF7739F0AA.png)

### Gourad/Phong Shading
Gourad shading: 
The light intensity is calculated per vertex, and then this value is interpolated and passed to each fragment for color computation.
> Majority of computation is offloaded to vertex shaders, making it fast, but lighting can look bad on low-poly models.

Phong shading:
The normal is obtained per vertex, and this value is interpolated and passed to each fragment to calculate the light intensity per fragment, then applied to color computation.
> Lighting is mostly performed on each fragment, lowering performance but improving the light quality

![](https://i.pcmag.com/imagery/encyclopedia-terms/gouraud-shading-_shading.fit_lim.size_1050x.gif)

## Normal Mapping
https://learnopengl.com/Advanced-Lighting/Normal-Mapping
Surface normals are used for lighting calculations.
They are usually per-vertex or per-pixel. Per-pixel normal values can be stored in a "normal map"

Normals help to calculate the angle of reflection of a texture surface. For a flat plane, the entire plane will have the same normal, thus it would make sense that it should look "smooth". For a bumpy wall, it would be expected that this wall would have some minor "bumps", and light bouncing off of this surface should not look flat. We can induce this effect using per-pixel normals that point in slightly different directions to make the surface not feel as "flat".
![](https://learnopengl.com/img/advanced-lighting/normal_mapping_surfaces.png)
When applied to an actual surface, it look like this:
![](https://learnopengl.com/img/advanced-lighting/normal_mapping_compare.png)

Ususally, raw model files (.fbx, .obj etc) should have per-vertex normals included. Per-vertex normals can be used to obtain per-fragment normals (blinn-phong), and then that would be used for lighting. However, custom bumpy surfaces like brick walls can't be interpolated from model per-vertex normals (how are you going to generate the normals along the seams and crevices?), so artists usually provide a normal map to get the accurate per-pixel lighting they want. These normal maps can be generated by their art/modelling software.
![](https://artisticrender.com/wp-content/uploads/2022/05/ObjectNormalExample.jpg)
### Types of Normal Maps
There are a few flavours when it comes to normal maps.
You could have your normal maps in different systems: tangent space, object space and world space.
The difference is the direction that these normals are pointing in, or how they are interpreted.
![](https://lh5.googleusercontent.com/proxy/nvwe6QpZJ4YJLaW65I5llM1Tvk6DLz6pt2FBVqLtFD6DPYo8GUmCKSltBJ8eOrr2fe8vBplwRiW8sd2Tv_Gv86vzfAeOO31OdZKlvut2pcJ1XnoG9g)
#### Tangent Space
![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Image_Tangent-plane.svg/1920px-Image_Tangent-plane.svg.png)
Tangent space maps always have the Z-axis pointing up away from the surface of the object (assuming a flat surface). Thus, the normal map will always have a blue-ish tint consistently across the whole map (because Z corresponds to B in a vector3).
![](https://learnopengl.com/img/advanced-lighting/normal_mapping_normal_map.png)
#### Object Space
Each normal has their axes aligned to the axes of the object itself. This is different from tangent maps, where each normal's axes are aligned according to the surface of wherever each pixel rests on.
#### World Space
Similar to object space, but instead of the axes aligning to the object, axes are aligned to the world.
#### What should you use
From [this thread](https://polycount.com/discussion/227725/difference-between-object-space-and-world-space-normals):
>Tangent space maps are more reusable (tiling textures for instance), and also work with skinned/deforming objects.
They are also more prone to artefacts and the lighting calculation is slightly more complex (but negligible)
For a non deforming asset with its own normal map, you could use object space.
For completely static object with its own normal map, you could use world (but no one do this).
Pretty much everybody use tangent.
#### More reading
https://blenderartists.org/t/tangent-vs-object-space-vs-world-space/456423
## Lighting calculations with Tangent Space
Normally, if we'd used world space or object space normals, computing the angle of reflection of a light ray is simple enough, as everything is in (almost) the same coordinate system.
Let's assume we are going with tangent space normals instead.
In order to use tangent normals for lighting, we'd need the light source and the normal to be in the same coordinate system. Right now, they are in different systems, world space and tangent space. Thus, we'd need to build a change-of-basis matrix to convert either the light or normal to the other space. For that, we'd need to obtain the 3 basis vectors to build the change-of-basis matrix, which are the Normal, Tangent and Bi-Tangent vectors. We'll need these vectors on a per-vertex basis.
### Calculating Tangent and Bi-Tangent
https://learnopengl.com/Advanced-Lighting/Normal-Mapping
Skippa skippa. Just note that this change of base matrix is called a TBN (Tangent, BI-tangent, Normal) matrix.
TLDR: Calculate it manually when importing your assets, or use the importer tool (e.g. assimp) to generate tangent and bi-tangents.
> If you're generating a custom mesh (e.g. generating terrain) you might have to do this calculation manually. This is usually implemented in tooling and not really for game runtime so that's a problem for next time.

After obtaining thsoe vectors, these per-vertex vectors should look something like that, witht the blue z axes pointing away from the surface instead of globally up:
![](https://learnopengl.com/img/advanced-lighting/normal_mapping_tbn_shown.png)
### Applying TBN in shaders
Our light and normals are still in different coordinate systems. But we now have a change-of-basis matrix!
There are 2 ways to go about computing lighting:
1. Convert everything into world space and compute in world space
2. Convert everything into tangent space and compute in tangent space

Hint: Computing in Tangent space is more efficient. Read more below.
#### 1. Computing in World Space
Need to convert the tangent space normal into world space normal. Light direction is already in world space.
Compute the TBN matrix per-vertex, then pass the info to the fragment shader. It should look something like this in glsl vertex shader:
```
// Inputs from host
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoords;
layout (location = 3) in vec3 aTangent;
layout (location = 4) in vec3 aBitangent;  

// outputs for later shader stages (frag shader)
out VS_OUT {
    vec3 FragPos;
    vec2 TexCoords;
    mat3 TBN;
} vs_out;

void main()
{
   [...]
   vec3 T = normalize(vec3(model * vec4(aTangent,   0.0)));
   vec3 B = normalize(vec3(model * vec4(aBitangent, 0.0)));
   vec3 N = normalize(vec3(model * vec4(aNormal,    0.0)));
   vs_out.TBN = mat3(T, B, N); // compute TBN matrix and pass to frag shader
}
```
In the fragment shader, you'd want to grab the normal from the texture map (still in tangent space), and then apply this TBN matrix to the tangent space normal. You'll get a world space normal. Now you can use this world space normal for lighting calculations!
```
// Input from vert shader (WARNING: Values are interpolated)
in VS_OUT {
    vec3 FragPos;
    vec2 TexCoords;
    mat3 TBN;
} fs_in;  

normal = texture(normalMap, fs_in.TexCoords).rgb;   // extract normals from normal map. Normal is in Tangent Space.
normal = normal * 2.0 - 1.0;                        // Re-adjust normals extracted from map to turn it from range [0, 1] to [-1, 1]
normal = normalize(fs_in.TBN * normal);             // Apply TBN matrix to tangent space normal, get world space normal.
```
> Note: When transferring data from vertex to fragment shader, values can't be passed 1 to 1. Instead, values passed from vertex shaders to fragment shaders are interpolated depending on their position in the triangle. This also means the values in your matrix are interpolated as well. This may cause the matrix to end up not being orthogonal when it reaches your fragment shader. You can try to re-orthogonalize the matrix by extracting 2 of the axes and then re-calculating the 3rd axis from cross-product-ing. Or you could use the matrix as-is but risk inaccurate normals.

#### 2. Computing in Tangent Space
In the vertex shader, we compute the TBN matrix as usual. But this time, we need an inverse matrix of TBN, as we're converting things the other way round. Due to the matrix being orthogonal, transposing the matrix is the same as inverting the matrix.

Now, instead of passing in the matrix to the fragment, we perform matrix multiplication in the vertex instead. What do we multiply? _Not_ the normal, because that's in the fragment and not vertex. We instead pre-compute all the values we need for lighting calculations first before sending it over to the fragment shader. 
This means that you should pre-compute:
- TangentLightPos
- TangentViewPos
- TangentFragPos

With these 3 values, and the tangent normal later, we can compute the lighting.
```
// VERTEX SHADER
out VS_OUT {
    vec3 FragPos;
    vec2 TexCoords;
    vec3 TangentLightPos;
    vec3 TangentViewPos;
    vec3 TangentFragPos;
} vs_out;

uniform vec3 lightPos;
uniform vec3 viewPos;
 
[...]
  
void main()
{    
    [...]
    mat3 TBN = transpose(mat3(T, B, N));
    vs_out.TangentLightPos = TBN * lightPos;
    vs_out.TangentViewPos  = TBN * viewPos;
    vs_out.TangentFragPos  = TBN * vec3(model * vec4(aPos, 1.0));
}  

// FRAG SHADER
void main()
{           
    vec3 normal = texture(normalMap, fs_in.TexCoords).rgb;
    normal = normalize(normal * 2.0 - 1.0);   
   
   // normalize values because interpolated values from fs_in may not be normalized anymore
    vec3 lightDir = fs_in.TBN * normalize(lightPos - fs_in.FragPos); 
    vec3 viewDir  = fs_in.TBN * normalize(viewPos - fs_in.FragPos);    
    [...]
}  
```
# Pipelines
## Vertex Shader Pipeline
## Mesh Shader Pipeline


# Shading
https://www.aortiz.me/2018/12/21/CG.html
## Forward Render
- Renders and shades each object in a single pass, lights and all.
- Each object gets shaded based on all lights that affect it (or in the scene). Will get very expensive if there are many lights.
- Handles transparency nicely, assuming the order is drawn from back to front. Blends the colors based on depth nicely.
- May be hard to handle some post processing effects like screen-space reflections, ambient occlusion, or complex multi-pass shaders due to being single-pass.

## Deferred Render
https://learnopengl.com/Advanced-Lighting/Deferred-Shading
### Deferred Shading
- Renders objects in 2 passes. 
- First pass: Geometry Pass. Stored in the gbuffer.
    - Gbuffer contains details such as color, depth, normals etc WITHOUT lighting. Basically all the data you'd need to do lighting.
    ![](https://learnopengl.com/img/advanced-lighting/deferred_g_buffer.png)
- Second pass: Deferred Shading Pass.
    - Lighting is computed per-pixed using data from each pixel in the gbuffer. Also consideres all lights in a scene, but does not do work on pixels from objects that are already covred. Reduces wasted lighting computation due to overdrawn pixels.
    ![](https://learnopengl.com/img/advanced-lighting/deferred_overview.png)
    - Filling gbuffers is not too bad, can fill them all in a single render pass by using multiple render targets (MRT)
    - Lighting computation is the same as in Forawrd rendering. It also opens up possibilities for more advanced lighting methods
- Post processing is applied after the final pass. Due to the existence of gbuffer, it's easier to implement many effects.
- Gbuffer results in more memory use, depending on number of buffers and the data precision requried by each buffer.
- Transparency won't work properly without additional funky workarounds, as the g-buffer only stores information of the top-most fragment, disallowing blending from happening.
    - Workaround: Add an additional Foward Render pass after deferred rendering, for objects that have transparency. Use the existing depth buffer from the gbuffer so that the foward rendered objects don't always render on top of all other objects
    - ![](https://learnopengl.com/img/advanced-lighting/deferred_lights_depth.png)

### Light Volumes
Calculate the radius that a point light can reach fragments. According to the maths, the intensity of light on a pixel from a light source should approach 0 but never reach 0, thus light has an "infinite" radius. But we can set a "dark enough" value to cut off this radius at a certain radius.

With a radius and the point light position, we can create a light volume (basically a sphere).
Eh the rest idk go and read it up: https://learnopengl.com/Advanced-Lighting/Deferred-Shading
If im not wrong, we render only fragments IN each light sphere by rendering the SPHERE itself to take advantage of the GPU's thread branching shennanigans. We don't actually render texture for the sphere, but we abuse the fact that when we "render" the sphere, we also touch the fragments that need to be shaded. So we run lighting on those. If these light volumes overlap, we blend the results together additively (whatever that means). Tada, lighting! Instead of iterating over every fragment on the screen and checking if it can be lit, we only touch those that we KNOW should be lit. Black magic.
![](https://learnopengl.com/img/advanced-lighting/deferred_light_volume_rendered.png)

### Deferred Lighting
Additional optimization on top of deferred shading.
TODO.

### Tile based deferred shading
Additional optimization on top of deferred shading
TODO.

## Forward+ Render
https://github.com/bcrusco/Forward-Plus-Renderer
TLDR:
- For every fragment, culls away potential light sources that never reach this fragment
- Instead of doing per-fragment, it checks and culls light sources per-frustum of 16x16 pixel tiles. The near/far planes of this frustum is determined by the min and max depth of all the fragments in this 16x16 tile.
    - 16x16=256 threads are used to compute the min/max depths for the near/far plane of the frustum, and the frustum is then created.
    - 16x16=256 threads can then be used in parallel for checking if each light source in the scene can reach this frustum or not. Keep checking 256 lights at a time until all lights in the scene are checked.
    - Visible light sources for every tile is stored in a shader storage buffer object (SSBO)
- Final shading happens in a second pass. For each frag, check which tile it belongs to, and extract the light sources to use from the SSBO. Tada, done.

## Tiled Shading
## Clustered Shading

# Culling
## Frustum Culling (CPU)
## Z Prepass
## HBZ
## Occlusion Culling (CPU)
## GPU Culling

# Lighting
## Static vs Dynamic Lights
## PBR
## Raytracing

# Shadows
## Shadow Map
https://en.wikipedia.org/wiki/Shadow_mapping
## Cascading Shadow Map

# Rendering Pipeline Architecture
Focuses on the CPU-GPU communication. Goal is to reduce overhead on data transfers, reduce amount of data being sent, and optimize GPU utilization.
## YOLO
Loop through each object and render each one with its own draw call. Many small and frequent draw calls, lots of data being sent, not good.
## GPU-Driven Rendering
Broad concept of using the GPU to handle more of the rendering workload that was traditionally handled by the CPU. Used for rendering huge scenes.

Implementations by others:
(Unity 6) GPU Resident Drawer
(Anvil) GPU Instance Renderer
### Batch Rendering
Grouping rendering commands into a single batch (draw call).
Batch needs to use the same state/resources in order to minimize swapping out resources and render state.

Objects are grouped into static and dynamic batches. Static batches are for non-moving objects, which means it can be reused without re-gerenating data. Dynamic batches are for objects that move around in the scene, thus the data needs to be updated every frame. Separating these 2 types open up areas for optimization.
#### GPU Instancing (Instance Rendering)
A form of batch rendering.
Renders multiple instances of the same object in a single draw call. Useful for many identical objects.
### Multi-Draw Indirect
Multiple draw commands get written to a buffer, and the CPU triggers a single draw call.
The GPU reads this buffer and performs multiple commands in one go.
### Bindless Textures

# Post Processing
## Anti-aliasing
### FSAA
### MSAA
## Motion Blur
## Bloom

# Power Efficient Rendering
https://www.jonpeddie.com/news/trends-and-forecasts-in-computer-graphics-power-efficient-rendering/
https://pdf.sciencedirectassets.com/280203/1-s2.0-S1877050923X00027/1-s2.0-S1877050923000698/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEHcaCXVzLWVhc3QtMSJGMEQCIGnpi5eyAATKRHCkbJ%2Fy43vPUXDq7AMJBsfpO%2BLMHZZZAiBPk6GbKnZSVWnbn28oh3%2FHXnQk%2FzS0%2FQnbvq9bUDNbgiq7BQig%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAUaDDA1OTAwMzU0Njg2NSIMOBRZ2OCKOeK4l2C0Ko8Fv7BHNx60lWiq96O4vovS%2FCoppvVdA2FqnItPFjqiFJsY9hjpK%2FztPdTUcgO5LNGL9XGjgbpODqDtNk2n%2B38CMehZlmk5gd1sGcQNKMx%2FanfkSeKZTXQfPVirOofDt%2FfJBhrwjrxqAtUOK3li%2BtHFc2QtIo%2Bgzc6L848i9J%2BeLTwKYGr7oAITAsEiVvtTVZ8R6pzVy4c1vZhv3WjpqdLKbDQ8SsSZYEAqOkE5Xc3JRS6LrXyOWooQiOIFWpdqfT2xTVzr60uCzG54UFRnUO0msaYdtxl80JUhbr0zU3IvJNLYz0MzPsqaYMSBRd2O4QvMu2lrPsxqErZXMNgx50Rd4KHRDHZYkc76e6R7n%2BDsEoCNwnKxquD7Wtckfms6K3xKCCjBGo2yci0A5p%2B7NVCJkHwYyg7pBVr0ushzzjTWAUykwE%2BE7C1qSPtFefmW5SZm3ceIe8%2BMr3I1v389UkQGJ7%2B2Cz0NaxUsBDyLR6huy4B85cFpcyprl8iHSrIRbOjgjNstD4ny0Q1xfcBrkz%2F%2FQ7nAqqxnCMhvFJu%2BUfaQU6nuGFLfXtfvvWTGKml8UPIN75h2%2BjOvIBjd4kI2cRR3bgnmQaHyP5wwlFefj5PqNesnmEN5b%2FZ7VH2OEP5Ixbd7IYB4xwyhNwkUviajX7e%2FHOy4jnHqMdcc4QPx84ksgYvuE8y5aCFxur9OC%2BHOtQBBSVIA18cJ3VSmmp9XDkw7rLbq4ziEG8CSVySxfTWEPGWVYhBpdPWhXzm7POsJ41xt39uTSo2pJ0H0qn%2BKvDB5NTywJTfwfZfgZTW5k2ZvHYSHofGAPJNd00EC0ToD%2FG7HGvtdrZTNPo9ZCqR5cuIZ5QKsyay3ISePuAiWrMWnOTDXm4q3BjqyAaxS%2FRfdOLruuvsfZ2jalggZOvD0D%2BmXzrykC3xE8TZPby8IZ4S28L0fwri%2F7XyjkUZFBN3JrpPZrCmhHLiSNoIhmr9hXERKCXSESusWPXQrKf0ShM%2BwHEazu1qL%2F3%2B0ebzNSvG%2BvkYEj77b6PnvJ89O%2F8E12Khd7vGS7MIRqzakXFAK27lPFfs2F4rOE4z3ozQ4%2BQoWnZHYVJ6Z8dxX2OrAOl4u%2B5mJE8EOwQ4Ab%2FWRSK8%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20240912T072406Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYRFR4CYX3%2F20240912%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=465e15e98b3b277a5fdb27d2e9da12f5edfb1ce6ad28a0d1a100cc9b9237ea4d&hash=c4d6654cb9f97b226dc5c8298599042de9e229a1456f9a2751dfdf69b1d79a12&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=S1877050923000698&tid=spdf-33e5cee7-ae02-427c-9833-546cad4248a6&sid=6171501676d33546139b74e24e9ca3f8ff37gxrqb&type=client&tsoh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&ua=1d035a03560153520a&rr=8c1e27e85fd19c65&cc=sg
Interesting things:
- Immediate mode rendering may be more power efficient depending on the amount of updates. A dynamically switching render mode can be most power efficient.
- Regular rendering techniques also apply here.