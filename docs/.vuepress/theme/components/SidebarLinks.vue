<template lang="pug">
ul.sidebar-links(v-if="items.length")
  li(v-for="(item, i) in items", :key="i")
    SidebarGroup(
      v-if="item.type === 'group'",
      :item="item",
      :open="i === openGroupIndex",
      :collapsable="item.collapsable || item.collapsible",
      :depth="depth",
      @toggle="toggleGroup(i)"
    )
    SidebarLink(v-else, :sidebarDepth="sidebarDepth", :item="item")
</template>

<script>
import SidebarGroup from "./SidebarGroup.vue";
import SidebarLink from "./SidebarLink.vue";
import { isActive } from "../util";

export default {
  name: "SidebarLinks",
  components: { SidebarGroup, SidebarLink },
  props: {
    items: {
      required: true,
      type: Array,
    },
    depth: {
      required: true,
      type: Number,
    },
    sidebarDepth: {
      default: 0,
      type: Number,
    },
  },
  data() {
    return {
      openGroupIndex: 0,
    };
  },
  watch: {
    $route() {
      this.refreshIndex();
    },
  },
  created() {
    this.refreshIndex();
  },
  methods: {
    refreshIndex() {
      const index = resolveOpenGroupIndex(this.$route, this.items);
      if (index > -1) {
        this.openGroupIndex = index;
      }
    },
    toggleGroup(index) {
      this.openGroupIndex = index === this.openGroupIndex ? -1 : index;
    },
    isActive(page) {
      return isActive(this.$route, page.regularPath);
    },
  },
};

function resolveOpenGroupIndex(route, items) {
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    if (
      item.type === "group" &&
      item.children.some((c) => c.type === "page" && isActive(route, c.path))
    ) {
      return i;
    }
  }
  return -1;
}
</script>

<style lang="scss" scoped>
.sidebar-group-items {
  li {
    .sidebar-link {
      position: relative;

      &::before {
        content: "";
        position: absolute;
        left: -15px;
        top: 7px;
        border: 1px solid var(--text-color);
        border-radius: 50%;
        height: 4px;
        width: 4px;
      }

      &.active {
        &::before {
          border-color: var(--accent-color);
          background-color: var(--accent-color);
        }
      }
    }
  }
}
</style>
