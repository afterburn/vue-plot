<script lang="ts">
import Vue from 'vue';
// Uncomment import and local "components" registration if library is not registered globally.
import { LineGraph } from '@/entry.esm';

export default Vue.extend({
  name: 'ServeDev',
  components: {
   LineGraph,
  },

  data() {
    return {
      data: [
        [0, 670.083],
        [63, 648.297],
        [126, 609.14],
        [189, 618.952],
        [252, 544.733],
        [315, 521.324],
        [378, 521.982],
        [441, 465.814],
        [504, 493.411],
        [567, 371.442],
        [630, 466.558],
        [693, 302.238],
        [756, 345.249],
        [819, 329.911],
        [882, 259.528],
        [945, 145.526],
        [1008, 110.93],
        [1071, 179.909],
        [1134, 186.363],
        [1197, 50],
      ],
    };
  },
});
</script>

<template>
  <div id="app">
    <div style="width: 200px; height: 80px; color: blue;">
      <LineGraph
        :data="data"
        :stroke-width="4"
        :smoothness="0.2"
      />
    </div>
  </div>
</template>
