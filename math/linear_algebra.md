2 problems

 - solving systems of equations
 - fitting some data with an equation

# dot product

**triangle inequality**: a sum of the size of vector a and vector b is greater or equal to the size of these two vectors summed.

# projections

**scalar projection** is the size of the "shadow" (the projection) of vector s on vector r. It's the size of the adjacent side of a triangle when you drop a perpendicular line from s to r. It's a number, a scalar. It's greater if the vectors are more aligned (parallel with each other), ie if the cosinus is greater. 

**vector projection** is the scalar projection of s on r multiplied by the normalized (size = 1) vector r. This means you get a vector in the direction of r, with the "correct" size (ie size of the "shadow"). It is a vector in that direction, with size = scalar projection.

# changing basis
A basis is a set of vectors that are not linear combinations of each other. Geometrically, this means they lie on different planes. They do not have to be orthogonal and normalized (size = 1), but it's easier if they are.

A vector can be described as a sum of its basis vectors.

You can describe vectors with other basis vectors. If you know the coordinates of the new vectors in the old basis, then you can use projections to calculate the coords in the new basis. This is only if the new basis has orthogonal vectors! Otherwise you need to use matrix operations.

 - calculate the vector projection of vector to the vectors in the new basis (in terms of the old basis!)
 - express the vector projections in the form `a * basis_vector`
 - these `a` terms express the vector in terms of its new basis!

# the "linear" in linear algebra
When we map from one basis to another the projection keeps the grid being evenly spaced and our original rules of vector addition and multiplication by a scalar still work. It doesn't warp or fold space, which is what the linear bit in linear algebra means geometrically. Things might be stretched or rotated or inverted, but everything remains evenly spaced and linear combinations still work.

# matrices

# inverse
$$A^-1 * A = I$$