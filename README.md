# Assignment for the case-structure lecture

## Goals

- Get familiarized with OpenFOAM's case files organization and purpose
- Master OpenFOAM file format

## Basic-level skills

Let's start by reviewing some commands shown in the lecture video:

> All these commands are meant to be run inside the docker container we created earlier!

This step is optional, but recommended: We start by creating (and switching to) a directory to hold all
our future OpenFOAM cases. You can skip this and create cases anywhere you like.
```bash
> mkdir -p $FOAM_RUN; run
```

Now, we copy a simple tutorial case (called "cavity") to this directory.
```bash
> cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity .
```

**Question:** Are you familiar with these $FOAM_RUN and $FOAM_TUTORIALS shell environment variables? 
If not, please go back and go through earlier lectures ...

List all files and directories (and go one level deeper into these directories) of the cavity case:
```bash
> ls cavity/*
```

The output of that command should look like:
```bash
cavity/0
  U  p

cavity/constant:
  polyMesh  transportProperties

cavity/system:
  controlDict  fvSchemes fvSolution
```

I want now to look inside each file (in this order) and answer the following questions:

### Main variables

We can see that there are pressure (p) and velocity (U) dictionaries in `cavity/0` directory.

1. What is the class of the `U` field?
2. What do you think `volScalarField` means (use your own judgment, no searching required)
   - [ ] A homogenous field (all values must have the same value)
   - [ ] A field of scalar, cell-based (possibly heterogeneous) values
   - [ ] A point-based (mesh-vertex-based) field
3. How a single 3D vector is represented in OpenFOAM files?
4. Based on 2., you can deduce what `volVectorField` is, right?
5. There is also a `surfaceScalarField`, which is similar to `volScalarField`, but face-based instead!
6. Give one more (potential) example of each class:
   - `volScalarField` => phase saturation (alpha), ...
   - `volVectorField` => position of mesh cell centers, ...
   - `surfaceScalarField`=> phase flux (phi), ...
7. Is the 3D coordinates of mesh points form a `volVectorField`?
8. Pick the right mathematical representation of the boundary condition for `U` 
   (as specified in the `cavity/0/U` file) for the `movingWall` patch
   - [ ] for each boundary face b, 
   
   <a href="https://www.codecogs.com/eqnedit.php?latex=\mathbf{U_b}&space;=&space;\begin{bmatrix}&space;1&space;\\&space;0&space;\\&space;0&space;\end{bmatrix}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mathbf{U_b}&space;=&space;\begin{bmatrix}&space;1&space;\\&space;0&space;\\&space;0&space;\end{bmatrix}" title="\mathbf{U_b} = \begin{bmatrix} 1 \\ 0 \\ 0 \end{bmatrix}" /></a>
   - [ ] for each boundary face b, 
   
   <a href="https://www.codecogs.com/eqnedit.php?latex=\mathbf{\nabla&space;U_b}&space;=&space;\begin{bmatrix}&space;1&space;\\&space;0&space;\\&space;0&space;\end{bmatrix}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mathbf{\nabla&space;U_b}&space;=&space;\begin{bmatrix}&space;1&space;\\&space;0&space;\\&space;0&space;\end{bmatrix}" title="\mathbf{\nabla U_b} = \begin{bmatrix} 1 \\ 0 \\ 0 \end{bmatrix}" /></a>
   
### The Mesh
   
> This particular tutorial uses a mesh-generation utility called `blockMesh`. Learning how this utility is supposed to work 
> is best postponed to a later lecture, but this section tries to recon the generated mesh

You can generate the mesh with the following command (run in cavity case directory):
```bash
> blockMesh
```

Based on the command output *only*,

> The following questions are based on the output of blockMesh in a Foam-Extend
> environment; you'll have trouble answering with Mainline OpenFOAM

1. How many mesh cells are there?
2. How many faces the mesh has
3. How many **boundary** faces are there
4. These boundary faces are assembled into 3 boundary patches, order the following face "groups"
   according to their starting face:
   - movingWall patch faces;
   - fixedWall patch faces;
   - frontAndBack patch faces;
   - Internal faces;
   
It's also good practice to check mesh quality immediately after generating the mesh:

```bash
> checkMesh
```
Based on the command output *only*,

1. Is the generated mesh a 1D, 2D, or 3D one?
2. Does `checkMesh` agree on using this mesh for OpenFOAM simulations?
3. What are Min/Max cell Volumes (in meter cubed)? Hint: look for "Min volume = ..." line.
4. Are there very small faces in the mesh?

### Simulation configuration

It's expected to see time-derivative, gradient, divergence, and laplacian terms in a PDE representing fluid flow.
`cavity/system/fvSchemes` contains few dictionaries to specify how each term in the equation should be treated.

1. What's the default scheme for 1st-order time-derivatives (ie. d/dt terms)?
2. Note that spacial discretization in this case relies mainly on gaussian integration with linear interpolations.
   List some of the equation terms that are explicitely mentioned in `cavity/system/fvSchemes`
   
> You can ignore interpolation and "snGrad" (surface-normal gradients) schemes for now.

After these schemes are used to produce algebraic equations for each mesh cell, the solver will try to
solve the generated matrices using a specified **linear solver**. Take a look at `cavity/system/fvSolution`
and answer the following questions:

> Ignore the PISO dictionary for now

3. You remember us having two main variables (p, U) in inital time directory, right?
   Which "linear solver" is associated with each solved-for variable?
   
> PCG: Preconditioned Conjugate, 
> Bi-PCG-Stab: Stabilized Bi-PCG, but these may differ accross OpenfOAM versions/forks

Now we get to the `cavity/system/controlDict` file,

4. You can see that the simulation time **starts from** `startTime`, which is defined as `startTime 0;`.
   How many simulation-time seconds (time units) should pass before the simlation is finished? Hint: `endTime`. 
5. Starting deltaT (timestep length)? is this deltaT modifiable at run-time (as the simulation progresses)?
6. The solver will write its results to new time directories each 20 timesteps. Identify the two keywords responsible
   for this behavior in `cavity/system/controlDict`
7. What is the class of the `controlDict` object?

At this point, you should be ready to meddle with the case, and we classify that as intermediate-skills.
To keep things clean you should remove the generated mesh (execute this in case directory; which is $FOAM_RUN/cavity):

> foamCleanTutorials

## Intermediate-level skills

### The mesh

We won't care for how the mesh is created in this section, we are interested in digging into some mesh files instead.
So, please, re-generate the mesh and check its quality.

The files we are interested in are located in `cavity/constant/polyMesh`. 
Start with `cavity/constant/polyMesh/points` file.

1. What is the class of the points coordinates? there is no "vol"
   in there because "vol*" stuff are defined at **cell centers**, not at cell vertices.
2. Aside from the file header, there is only a list of 882 vectors in this file; each vector
   representing 3D coordinates of a cell vertex. See how OpenFOAM lists are specified?
   `size (... elements here ...)`. What are the 3D coords of the point whose index is 0 (the first one)?
   what about the last one (index 881)?

Switch to `cavity/constant/polyMesh/boundary`.

3. Boundary patches are specified as:
   - [ ] a dictionary of face lists
   - [ ] a list of dictionaries
   - [ ] an array of dictionaries
   
4. What is the index of "fixedWalls" patch? (Indexing also starts from 0 here)

Check `cavity/constant/polyMesh/faces`

5. A mesh face is defined using vertex **labels**. `4(0 1 2 3)` means a quad of vertices 0,1,2,3 
   (in manifold ordering). Assume all faces defined in this file are quads, can deduce cells geometry?
   Check it in `checkMesh` output.

To illustrate the importance of checking mesh quality, let's modify a mesh file and see what happens:

Find the `fixedWalls.nFaces` keyword in `cavity/constant/polyMesh/boundary` and change its value from 60 to 61.
This virtually means we are increasing the number of faces for this patch without taking into account other patches 
(obviously, erroneous action).

> This notation: `fixedWalls.nFaces` means look for the keyword (or sub-dictionary) "nFaces" inside the dictionary
> "fixedWalls".

Now run `checkMesh` and look for lines in thet output starting with "***" 
(The following also displays lines that start with "Mesh"):

```bash
checkMesh | grep -e "\*\*" -e "^Mesh"
```

Cool, it says "Boundary definition is in error".
But what is the name of the patch the tool thinks is problematic? what is its index?
Can you make sense of the line saying: **"The patch should start on face no 841 and the patch specifies 840."**?

But hey, `checkMesh` still says it's OK to proceed with the simulation, so we'll do just that; run:
```bash
icoFoam
```

`icoFoam` is the suitable solver for this particular case. Note that solver runs fine, with continuity errors of
order 1e-20 and everything!

What I want you now to do is to visualize the generating mesh using ParaView:

> Note that there are much better ways to do this, but we'll live with this method for now :smile:

```bash
# On the server (inside docker container, in cavity directory)
# Create an empty foam file (For ParaView)
> touch cavity.cavity
# Compress the case
> tar -czvf myCavity.tar.gz .
# Upload the case to transfer.sh (for example)
> curl --upload-file ./myCavity.tar.gz https://transfer.sh/myCavity.tar.gz
# You'll recieve a link to your file on transfer.sh
```

Download the compressed case to your machine, unpack it, and load `cavity.foam` into ParaView.

There you go, errors?

**The end face number 840 of patch fixedWalls is not consistent with the start face number 840 of patch frontAndBack**

ParaView explicitely tells us face whose index is 840 belongs to both patches! the OpenFOAM solver we used had no problem
running in such shady condition.

**Things to take:**
- OpenFOAM has this nice feature of "**what comes last wins**"!! Our configuration made face 840 belong to two patches but 
  it appears OpenFOAM allocates the face to the last patch (frontAndBack) and we were lucky it was the correct action.
- Don't take liberty in modifying case files carelessly, you'll cause yourself some serious headacke.

Changing that 61 to 60 in `constant/polyMesh/boundary` again allows for result visualization. No difference in the solution
is noticed.

> Don't forget to clean up the case: `foamCleanTutorials`

### Macro expansion in OpenFOAM file

Inside OpenFOAM dictionaries you can use a variable multiple time if you define it a keyword:

```cpp
someVar     0;
someDict
{
  someKeyword  $someVar;
}
```
Then, `someDict.someKeyword` would have the value of 5.

In `cavity/0/U` dictionary, the keyword `internalField` is already defined and has a value of `uniform (0 0 0)`.
Use it to assign the same value to the `boundaryField.fixedWalls.value` keyword.
