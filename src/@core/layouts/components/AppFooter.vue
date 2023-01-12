<template>
  <p class="clearfix mb-0">
    <router-link
      v-ripple.400="'rgba(113, 102, 240, 0.15)'"
      class="float-md-right"
      to="/coffee"
    >Kalia Network
      <span
        v-ripple.400="'rgba(113, 102, 240, 0.15)'"
        variant="outline-primary"
      >
        <feather-icon
          icon="HeartIcon"
          size="21"
          class="text-danger stroke-current"
        />
      </span>
    </router-link>
  </p>
</template>

<script>
import Ripple from 'vue-ripple-directive'

export default {
  name: 'AppFooter',
  directives: {
    Ripple,
  },
}
</script>
