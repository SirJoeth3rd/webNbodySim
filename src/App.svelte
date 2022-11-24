<script>
  import Gsim from './Gsim.svelte';
</script>

<div class="content">
<hr>
<h1>N-Body Simulation</h1>

<Gsim/>

<h2>What is a nbody simulation</h2>


<h2>Explanation of the Algorithm</h2>
<p>
  The simulation uses the Barnes-Hut algorithm to calculate the force
  between each particle. The naive approach is to use a double for loop
  in which you loop over each particle and then loop over each particle
  again summing the calculated forces as you go. This approach has a
  complexity of N&#178;. The Barnes-Hut algorithm in contrast has a complexity
  on average of N log(N). It works by recursively subdividing the space into
  a tree where each node represents some volume in the total space with a
  center of mass and total mass of all the stars contained inside the volume.
  Each leaf of the tree represents a single particle. After the tree has been
  built each particles force is calculated by traversing said tree, for each
  node encountered the volume of the node is divided by the distance to the
  node and if it is lower then some arbitrary constant
  (called the theta tolerance) then the force on the particle from the node
  is calculated as though it is a single big particle with a mass equal to
  the sum of the masses of the particles contained in the node and a position
  equal to the center of mass of the node. If the ratio of volume to distance
  of the node is less than theta tolerance the force calculation function is
  called recursively on every child node of said node. 
</p>
  
</div>

<style>
  :root {
    --yellow: #fedaba;
    --silver:#d4d4dc;
    --grey:#393f4d;
    --slate:#1d1e22;
  }

  div.content {
    width:960px;
  }

  h1,h2 {
    color: var(--yellow);
    position: relative;
  }
  p {
    position: relative;
    color: var(--yellow);
    width: 760px;
    margin: 0 auto;
  }
  :global(body) {
    background-color: var(--slate);
  }
  hr {
    width: 760px;
  }
</style>
