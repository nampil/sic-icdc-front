<template>
  <v-app>
    <core-filter/>

    <core-toolbar/>

    <core-drawer/>

    <core-view/>
  </v-app>
</template>
<script>
import { mapState } from 'vuex'
export default {}
</script>

<style lang="scss">
@import '@/styles/index.scss';

/* Remove in 1.2 */
.v-datatable thead th.column.sortable i {
  vertical-align: unset;
}
</style>
