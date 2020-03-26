<template>
  <div>
    <div
      :id="itemId"
      class="item-wrapper"
      @click="click()"
    >
      <div
        class="item"
        :class="getItemClasses()"
      >
        <slot
          name="itemBadge"
          :item="item"
          :emptyItem="emptyItem"
        ></slot>
        <span
          v-if="item.event && item.owned == null && showEventBadge"
          class="badge badge-pill badge-item badge-clock"
        >
          <span
            class="svg-icon inline clock"
            v-html="icons.clock"
          ></span>
        </span>
        <span
          v-if="item.locked"
          class="svg-icon inline lock"
          v-html="icons.lock"
        ></span>
        <span
          v-if="item.isSuggested"
          class="suggestedDot"
        ></span>
        <div class="image">
          <div
            v-once
            :class="item.class"
          ></div>
          <slot
            name="itemImage"
            :item="item"
          ></slot>
        </div>
        <div
          class="price d-flex align-items-center justify-content-center"
          :class="currencyClass"
        >
          <span
            v-once
            class="svg-icon inline icon-16"
            v-html="icons[currencyClass]"
          ></span>
          <span
            v-once
            class="price-label"
            :class="currencyClass"
          >{{ getPrice() }}</span>
        </div>
      </div>
    </div>
    <b-popover
      v-if="showPopover"
      :target="itemId"
      triggers="hover"
      :placement="popoverPosition"
    >
      <slot
        name="popoverContent"
        :item="item"
      >
        <equipmentAttributesPopover
          v-if="item.purchaseType === 'gear'"
          :item="item"
        />
        <div
          v-else-if="item.purchaseType === 'quests'"
          class="questPopover"
        >
          <h4 class="popover-content-title">
            {{ item.text }}
          </h4>
          <questInfo :quest="item" />
        </div>
        <div v-else>
          <h4
            v-once
            class="popover-content-title"
          >
            {{ item.text }}
          </h4>
          <div
            v-if="showNotes && item.key !== 'armoire'"
            v-once
            class="popover-content-text"
          >
            {{ item.notes }}
          </div>
          <div
            v-if="showNotes && item.key === 'armoire'"
            class="popover-content-text"
          >
            {{ item.notes }}
          </div>
        </div>
        <div
          v-if="item.event"
          class="mt-4"
        >
          {{ limitedString }}
        </div>
      </slot>
    </b-popover>
  </div>
</template>

<style lang="scss" scoped>
  @import '~@/assets/scss/colors.scss';

  .item-wrapper {
    z-index: 10;
  }

  .item {
    height: 7.5rem;
    width: 94px;
    border-radius: 4px;
    background-color: $white;
    box-shadow: 0 1px 3px 0 rgba($black, 0.12), 0 1px 2px 0 rgba($black, 0.24);
  }

  .item:not(.locked) {
    cursor: pointer;
  }

  .image {
    margin: 12px 13px;
  }

  .price {
    height: 1.75rem;
    width: 94px;
    margin-left: -1px;
    margin-right: -1px;
    border-radius: 0px 0px 4px 4px;

    &.gems {
      background-color: rgba($green-100, 0.15);
    }

    &.gold {
      background-color: rgba($yellow-100, 0.15);
    }

    &.hourglasses {
      background-color: rgba($blue-50, 0.15);
    }

    .svg-icon {
      margin-right: 4px;
    }
  }

  .price-label {
    height: 16px;
    font-family: Roboto;
    font-size: 12px;
    font-weight: bold;
    line-height: 1.33;

    &.gems {
      color: $green-1;
    }

    &.gold {
      color: $yellow-1;
    }

    &.hourglasses {
      color: $blue-1;
    }
  }

  span.svg-icon.inline.lock {
    height: 12px;
    width: 10px;
    position: absolute;
    right: 8px;
    top: 8px;
    margin-top: 0;
  }

  span.badge.badge-pill.badge-item.badge-clock {
    height: 24px;
    width: 24px;
    background-color: $purple-300;
    position: absolute;
    left: -8px;
    top: -12px;
    margin-top: 0;
    padding: 4px;
  }

  span.svg-icon.inline.clock {
    height: 16px;
    width: 16px;
  }

  .suggestedDot {
    width: 6px;
    height: 6px;
    background-color: $suggested-item-color;
    border-radius: 4px;

    position: absolute;
    right: 8px;
    top: 8px;
    margin-top: 0;
  }

  .icon-48 {
    width: 48px;
    height: 48px;
  }
</style>

<script>
import uuid from 'uuid';

import moment from 'moment';
import svgGem from '@/assets/svg/gem.svg';
import svgGold from '@/assets/svg/gold.svg';
import svgHourglasses from '@/assets/svg/hourglass.svg';
import svgLock from '@/assets/svg/lock.svg';
import svgClock from '@/assets/svg/clock.svg';

import EquipmentAttributesPopover from '@/components/inventory/equipment/attributesPopover';

import QuestInfo from './quests/questInfo.vue';


import seasonalShopConfig from '@/../../common/script/libs/shops-seasonal.config';

export default {
  components: {
    EquipmentAttributesPopover,
    QuestInfo,
  },
  props: {
    item: {
      type: Object,
    },
    price: {
      type: Number,
      default: -1,
    },
    emptyItem: {
      type: Boolean,
      default: false,
    },
    highlightBorder: {
      type: Boolean,
      default: false,
    },
    popoverPosition: {
      type: String,
      default: 'bottom',
    },
    showPopover: {
      type: Boolean,
      default: true,
    },
    showEventBadge: {
      type: Boolean,
      default: true,
    },
  },
  data () {
    return Object.freeze({
      itemId: uuid.v4(),
      icons: {
        gems: svgGem,
        gold: svgGold,
        lock: svgLock,
        hourglasses: svgHourglasses,
        clock: svgClock,
      },
    });
  },
  computed: {
    showNotes () {
      if (['armoire', 'potion'].indexOf(this.item.path) > -1) return true;
      return false;
    },
    currencyClass () {
      if (this.item.currency && this.icons[this.item.currency]) {
        return this.item.currency;
      }
      return 'gold';
    },
    limitedString () {
      return this.item.owned === false ? ''
        : this.$t('limitedOffer', { date: moment(seasonalShopConfig.dateRange.end).format('LL') });
    },
  },
  methods: {
    click () {
      this.$emit('click', {});
    },
    getPrice () {
      if (this.price === -1) {
        return this.item.value;
      }
      return this.price;
    },
    getItemClasses () {
      return {
        'item-empty': this.emptyItem,
        'highlight-border': this.highlightBorder,
        suggested: this.item.isSuggested,
        locked: this.item.locked,
      };
    },
  },
};
</script>
