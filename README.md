# Description of the project

This is a generic n body simulation, I did it to investigate using svelte, typescript and threejs. All three of these were new to me before 
I started so this code is likely ugly etc. Considering the very few resources for svelte cubed this could also be usefull in that regard for someone 
stumbling across it. 

# The algorithm

I used the Barnes-Hut algorithm, a recursive n log(n) algorithm that subdivides the space and places every particle into a tree structure that 
keeps track of the space in which the particle is located. This allows for optimization by considering far away particles as a single entity with
mass and position calculated for the space said particles is contained in. You can read more on that at [wikipedia](https://en.wikipedia.org/wiki/Barnes%E2%80%93Hut_simulation). Something potentially intersting in the version of this algorithm that I made is that the total space of a node is dynamically
updated as particles are inserted into said node all nodes the particle moves through when it is placed in the tree. From what I have seen in other implimentations a fixed space is usually subdivided. The difference being that without any further work this algorithm works for sets of particles with arbitary volumes. 

# The tools used

The project is used mostly to investigate some front end technologies with which I am not familiar. As I have said  before I used Svelte as the front end
framework with typescript for and threejs for the 3d rendering of the scene. Although later I switched over to using svelte-cubed to interact with three-js. 

# The code

If you want to investigate the code it is under src/Gsim.svelte, that is the main component that implements the algorithm and makes the scene. App.svelte just makes the rest of the html page. 

# To run this project

It should be as simple as cloning the repo and doing a `npm i`, 'npm run dev' and visiting 127.0.0.1:5000. 
