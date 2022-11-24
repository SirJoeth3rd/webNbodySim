<div class="scene-container">
  <SC.Canvas>
    {#each stars as star}
      <SC.Mesh
	geometry = {star.mesh.geometry}
	material = {star.mesh.material}
	position = {[
        star.mesh.position.x,
        star.mesh.position.y,
        star.mesh.position.z
	]}
	/>
      {/each}
      <SC.PerspectiveCamera position={[0,0,30]}/>
      <SC.OrbitControls/>
      <SC.AmbientLight color={new THREE.Color('white')}/>
  </SC.Canvas>
  <button on:click={invertAnimation}>
    {animate ? 'stop' : 'start'}
  </button>
</div>

<style>
  .scene-container {
    position: relative;
    width: 75%;
    max-width: 700px;
    height: 400px;
    margin: 0 auto;
    border-width: 2px;
    border-style: solid;
    border-color: var(--silver);
    margin-top:5px;
  }
  button {
    color: white;
    position: relative;
  }
</style>

<script lang="ts">
  //here comes the actual code which talks with above
  let stars = init_stars(80);

  //flag used to send messages to animation loop
  var animate = true;

  function invertAnimation () {
    animate = !animate;
  }
  
  //the big boy animation loop
  SC.onFrame(()=>{
    if (animate) {stars = move_via_gravity(stars,0.01)}
  });

  //the local implementation of move_via_gravity
  import * as THREE from 'three';
  import * as SC from 'svelte-cubed';
  
  type Vec3 = THREE.Vector3;
  type StarMesh = THREE.Mesh<THREE.SphereGeometry, THREE.MeshPhongMaterial>;

  type Star = {
    velocity: Vec3;
    accelaration: Vec3;
    mesh: StarMesh; //this already contains position information
  }

  function starMesh() {
    return new THREE.Mesh(
      new THREE.SphereGeometry(0.25, 24, 24),
      new THREE.MeshPhongMaterial({ color: 0xffffff, depthTest: false })
    )
  }
  
  function init_stars(N: number): Star[] {
    const stars = new Array(N)
	  .fill()
	  .map(() => {
	    const [x, y, z] = Array(3)
		  .fill(0)
		  .map(() => THREE.MathUtils.randFloatSpread(100));
	    let mesh = starMesh();
	    mesh.position.set(x, y, z);
	    let velocity = new THREE.Vector3(0, 0, 0);
	    let accelaration = new THREE.Vector3(0, 0, 0);
	    return {
              velocity: velocity,
              mesh: mesh as StarMesh,
              accelaration: accelaration
	    } as Star;
	  });

    //sets up initial accelaration values
    stars.forEach((star_a) => {
      stars.forEach((star_b) => {
	if (star_a != star_b) {
          const pos_a = star_a.mesh.position;
          const pos_b = star_b.mesh.position;
          const dir = (new THREE.Vector3).subVectors(pos_b, pos_a);
          star_a.accelaration.addScaledVector(dir.normalize(),
					      1.0 / (dir.length() ** 2))
	}
      })
    })
    return stars;
  }

  type Children = [Node, Node, Node, Node, Node, Node, Node, Node]
  type dir3D = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7
  type Space = { u: Vec3, d: Vec3 } | undefined

  type Node = {
    star?: Star;
    children?: Children;
    has_children: boolean;
    center_of_mass: Vec3;
    total_mass: number
    space: Space
  }

  function init_node(): Node {
    return {
      star: undefined, children: undefined, has_children: false,
      center_of_mass: new THREE.Vector3(0, 0, 0), total_mass: 0,
      space: undefined
    };
  }

  function node_make_children(n: Node) {
    n.children = Array(8)
      .fill(null)
      .map(init_node) as Children;
    n.has_children = true;
  }

  function octant(dest: Vec3, source: Vec3): dir3D {
    const tempvec = (new THREE.Vector3()).subVectors(dest, source);
    let dir = 0;
    if (tempvec.x > 0) { dir += 1 }
    if (tempvec.y > 0) { dir += 2 }
    if (tempvec.z > 0) { dir += 4 }
    console.log(dir);
    return dir as dir3D;
  }

  function dir3DToBoolTriplet(dir: dir3D): [boolean, boolean, boolean] {
    return [(dir & 1) == 1, (dir & 2) == 1, (dir & 4) == 1];
  }

  function subspace(n: Node, dir: dir3D): Space {
    if (n.space == null) { throw "cannot get subspace from nothing" }
    const vu = new THREE.Vector3(0, 0, 0); //the [1,1,1] corner
    const vd = new THREE.Vector3(0, 0, 0); //the -[1,1,1] corner
    const dirBools = dir3DToBoolTriplet(dir);

    vu.z = dirBools[0] ? n.space.u.z : n.center_of_mass.z;
    vd.z = dirBools[0] ? n.center_of_mass.z : n.space.d.z;

    vu.y = dirBools[1] ? n.space.u.y : n.center_of_mass.y;
    vd.y = dirBools[1] ? n.center_of_mass.y : n.space.d.y;

    vu.x = dirBools[2] ? n.space.u.x : n.center_of_mass.x;
    vd.x = dirBools[2] ? n.center_of_mass.x : n.space.d.x;

    return { u: vu, d: vd };
  }


  //gets the child of a node by numbered direction
  //is also the most convenient place to assign a node some space
  function get_node_child(n: Node, dir: dir3D): Node | null {
    if (n.children != undefined) {
      const child = n.children[dir];
      if (child.space == undefined) {
	child.space = subspace(n, dir);
      }
      return child;
    } else {
      return null;
    }
  }

  //if a star updates a nodes space is the function used to do it
  function update_node_space(n: Node, s: Star) {
    if (n.space == undefined) { return }
    const pos = s.mesh.position;
    const [d, u] = [n.space.d, n.space.u];
    if (d.z > pos.z) { d.z = pos.z }
    if (u.z < pos.z) { u.z = pos.z }
    if (d.y > pos.y) { d.y = pos.y }
    if (u.y < pos.y) { u.y = pos.y }
    if (d.x > pos.x) { d.x = pos.x }
    if (u.x < pos.x) { u.x = pos.x }
  }


  //update node center of mass and total mass
  function update_node_properties(n: Node, s: Star) {
    let m = n.total_mass + 1; //can do because every star has same mass
    n.center_of_mass.multiplyScalar(n.total_mass)
      .add(s.mesh.position)
      .multiplyScalar(1 / m);
    update_node_space(n, s);
    n.total_mass = m;
  }

  //the tree building function, meant to be called on every star in stars
  function insert_star(n: Node, s: Star) {
    if (n.has_children) {
      update_node_properties(n, s);
      insert_star(get_node_child(n, octant(
	n.center_of_mass, s.mesh.position)) as Node, s);
    } else {//
      if (n.star != undefined) {
	update_node_properties(n, s);
	node_make_children(n); //children gaurenteed
	insert_star(get_node_child(n, octant(
          n.center_of_mass, n.star.mesh.position)) as Node, n.star);
	insert_star(get_node_child(n, octant(
          n.center_of_mass, s.mesh.position)) as Node, s);
	n.star = undefined;
      } else {
	update_node_properties(n, s)
	n.star = s;
      }
    }
  }

  //add field force vector produced by m at p to f at positiion mp
  function add_field(f: Vec3, m: number, p: Vec3, mp: Vec3) {
    const G_ = 0.1;
    const dir: Vec3 = (new THREE.Vector3()).subVectors(p, mp);
    const strenght = m / (dir.length() ** 2);
    if (strenght == Infinity) { throw "Infinite field generated" }
    f.addScaledVector(dir.normalize(), G_ * m / (dir.length() ** 2)); //OVERFLOW
  }

  function ratio(n: Node, s: Star): number {
    if (n.space == undefined) { throw "cannot get ratio of empty space" };
    const volume = Math.abs((n.space.u.z - n.space.d.z)
			    * (n.space.u.y - n.space.d.y)
			    * (n.space.u.x - n.space.d.x));
    return volume / n.center_of_mass.distanceTo(s.mesh.position);
  }

  //give the root node of a tree and it will cacl the
  //gravitional field of a particle and store it in f
  function calculate_field(n: Node, s: Star, theta: Number, f: Vec3) {
    if (n.star != undefined) {
      if (n.star != s) {
	add_field(f, 1, n.star.mesh.position, s.mesh.position);
      }
    } else if (n.total_mass == 0) {
      //for emoty children
      return
    } else if (ratio(n, s) < theta) {
      add_field(f, n.total_mass, n.center_of_mass, s.mesh.position);
    } else {
      if (n.children != undefined) {
	n.children.forEach((child) => {
          calculate_field(child, s, theta, f);
	});
      }
    }
  }


  //given the force field at star, update it's position
  function move_star(s: Star, f: Vec3, dt: number) {
    const G = 0.00000001;
    //since our mass is one accelaration is just equal to force
    s.mesh.position
      .addScaledVector(s.velocity, dt)
      .addScaledVector(s.accelaration, 0.5 * (dt ** 2));
    s.velocity
      .addScaledVector
    (
      s.accelaration.addVectors(s.accelaration, f),
      0.5 * dt * G
    );
  }

  //creates the three strucure by calling mapping insert_star over list of stars
  function create_tree(star_list: Star[]): Node {
    let treetop = init_node();
    treetop.space = {
      u: new THREE.Vector3(0, 0, 0),
      d: new THREE.Vector3(0, 0, 0)
    }
    star_list.forEach((star) => {
      insert_star(treetop, star);
    });
    return treetop;
  }

  function move_via_gravity(all_stars: Star[], dt: number) {
    let treetop = create_tree(all_stars);

    all_stars.forEach((star) => {
      let f = new THREE.Vector3(0, 0, 0);
      calculate_field(treetop, star, 0.5, f);
      move_star(star, f, dt);
    });

    return all_stars;
  }

</script>


