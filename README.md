# CS488 Final Project: Grid-Accelerated Sand Pile Simulation

Name: Yutong Wu  

## Project Description

This project implements a simplified physics-based sand pile simulation in the CS488 framework. The simulation emits sand particles into a small scene, lets them fall under gravity, collide with a ground plane and a sloped ramp obstacle, interact with nearby particles, settle into piles, and generate a reconstructed height-field surface.

The project focuses only on sand. This keeps the scope specific and allows the implementation to include sand-related behavior such as friction, settling, slope stabilization, and angle-of-repose relaxation.

The final result is an interactive particle simulation where sand falls into the scene, slides over a ramp, accumulates near the ground, and forms a visible pile surface.

## Compilation

I compiled and tested the project on macOS using CMake.

Build commands:

    cmake -S . -B build -Dglew-cmake_BUILD_SHARED=OFF -DONLY_LIBS=ON
    cmake --build build -j8

Some warnings related to GLEW_STATIC redefinition and deprecated sprintf may appear on macOS. These warnings were also present in the base framework and do not prevent the project from compiling or running.

## Running

Run the program with:

    ./build/CS488

The final project entry point is FinalProject(argc, argv) in src/main.cpp.

Press uppercase F in the application window to start and stop GIF recording. The program saves GIF files with names like output###.gif.

## Implemented Features

### Objective 1: Sand Particle Emitter

The program contains a rectangular sand emitter near the top of the scene. Particles are initialized with randomized position, velocity, radius, and age. The emitter continuously adds new active particles until the particle budget is filled.

Relevant implementation:
- Particle::emitFromSandSource
- ParticleSystem::emitParticles

### Objective 2: Particle State and Sand Parameters

Each particle stores position, previous position, velocity, radius, age, active state, and settled state. The simulation uses Verlet-style position integration with gravity. Sand behavior is controlled through low restitution, friction, damping, and settling rules.

Relevant implementation:
- Particle
- Particle::integrate
- Particle::setVelocity

### Objective 3: Uniform Spatial Grid

A uniform spatial grid is used to accelerate local particle collision queries. Instead of checking every pair of particles globally, active particles are inserted into grid cells based on position. Each particle is then compared only with particles in nearby cells.

Relevant implementation:
- gridResolution
- gridCells
- ParticleSystem::buildSpatialGrid
- ParticleSystem::resolveSandParticleCollisions

### Objective 4: Local Particle-Particle Collision

Particles are treated as small spheres for collision. When two particles overlap, they are separated along the direction between their centers. Their velocities are adjusted with damping to reduce interpenetration and unstable bouncing.

Relevant implementation:
- ParticleSystem::resolveSandParticlePair
- ParticleSystem::resolveSandParticleCollisions

### Objective 5: Ground Collision and Friction

Particles collide with the ground plane and the side boundaries of the simulation box. Ground collision uses position correction, low restitution, and tangential friction so particles can slide and then slow down.

Relevant implementation:
- ParticleSystem::resolveGroundAndBoxCollision

### Objective 6: Ramp Obstacle Collision

The scene contains a visible sloped ramp made of two triangles. Particles collide with the ramp using a geometric plane test. When a particle penetrates the ramp plane, it is pushed out along the ramp normal and its velocity is modified to keep tangential sliding while removing most inward velocity.

Relevant implementation:
- ParticleSystem::resolveRampCollision
- ramp visualization code inside ParticleSystem::updateMesh

### Objective 7: Settling and Pile Formation

Particles can become settled when they are old enough, slow enough, and supported by the floor, ramp, or already settled particles. Settled particles stop moving and contribute to the sand pile. This reduces jitter and helps the sand form a stable pile.

Relevant implementation:
- ParticleSystem::updateSettledState

### Objective 8: Height-Field Surface Reconstruction

The project reconstructs a continuous sand surface from settled particles. The x-z domain is divided into a height field. Settled particles contribute to nearby height cells, the height field is smoothed, and the resulting surface is converted into triangles. This makes the sand pile appear more continuous than individual particles alone.

Relevant implementation:
- heightFieldResolution
- heightField
- smoothedHeightField
- ParticleSystem::updateSandSurfaceMesh

### Additional Feature: Angle-of-Repose Relaxation

The height-field surface includes a simplified angle-of-repose relaxation step. If neighboring height cells differ by more than the allowed slope for sand, height is transferred from the higher cell to the lower cell. This approximates the tendency of sand piles to stabilize below a maximum slope.

Relevant implementation:
- angleOfReposeDegrees
- angle-of-repose relaxation in ParticleSystem::updateSandSurfaceMesh

### Additional Feature: Grid Debug Visualization

The project includes a grid-cell debug visualization where particles are assigned different sand colors based on their spatial grid cell. This helps demonstrate that the spatial grid is being used for local collision queries.

Relevant implementation:
- grid-cell material assignment in ParticleSystem::updateMesh

## Output GIFs

The following GIFs were recorded during development:

- milestone1_sand_fall.gif: basic sand falling and accumulation.
- milestone2_spatial_grid.gif: spatial grid collision implementation.
- milestone2b_grid_debug.gif: grid-cell debug visualization.
- milestone3_ramp_obstacle.gif: ramp obstacle collision and sliding.
- milestone4_surface_reconstruction.gif: reconstructed sand pile surface.
- milestone5_angle_of_repose.gif: angle-of-repose relaxation on the reconstructed surface.

Older backup GIFs are not required for final submission.

## Implementation Notes

The simulation is not a full continuum sand solver. Instead, it uses a simplified particle-based model with spatial grid acceleration, local sphere collisions, friction, settling, height-field reconstruction, and angle-of-repose relaxation. This keeps the project feasible within the course framework while still demonstrating graphics-related simulation and visualization techniques.

The reconstructed surface is generated from settled particles only. This avoids floating surface patches caused by particles that are still falling through the air.

The ramp and ground are visualized as triangle geometry in the same mesh used for the particles and reconstructed surface.

## Known Limitations

The particle simulation is approximate and not a full physically accurate granular material solver.

The height-field surface is useful for visualization, but it is an approximation of the pile surface and may not exactly match every individual particle.

The angle-of-repose relaxation is applied to the reconstructed height field rather than directly to the particles themselves.

The project uses a fixed scene, fixed particle count, and fixed material parameters.

## Final Objective List

1. Implement a sand particle emitter with randomized particle initialization.
2. Implement per-particle state for active, moving, and settled sand particles.
3. Implement a uniform spatial grid for local neighbor search.
4. Implement local particle-particle collision using grid-based neighbor queries.
5. Implement ground and box collision with friction and low restitution.
6. Implement collision with a visible ramp obstacle.
7. Implement settling rules for stable pile formation.
8. Implement height-field surface reconstruction with angle-of-repose relaxation.
