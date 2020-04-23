<template>
  <div
    class="badge-pin align-items-center justify-content-center p-1"
    :class="{'pinned': pinned}"
  >
    <div
      class="svg-icon color"
      v-html="icons.pin">
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import '~@/assets/scss/colors.scss';

  .badge-pin {
    height: 1.5rem;
    width: 1.5rem;
    box-shadow: 0 1px 3px 0 rgba($black, 0.12), 0 1px 2px 0 rgba($black, 0.24);
    background-color: $white;
    border-radius: 100%;
    color: $gray-200;
    transition: none;
    display: flex;
    cursor: pointer;

    &:hover:not(.pinned) {
      color: $purple-300;
    }

    &.pinned {
      background-color: $purple-300;
      color: $white;
    }
  }

</style>

<script>
import svgPin from '@/assets/svg/pin.svg';

export default {
  props: {
    pinned: {
      type: Boolean,
    },
  },
  data () {
    return Object.freeze({
      icons: {
        pin: svgPin,
      },
    });
  },
};
</script>
