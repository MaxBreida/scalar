<script setup lang="ts">
import { HttpMethod } from '@scalar/api-client'
import { type Icon, ScalarIcon, ScalarIconButton } from '@scalar/components'

import { scrollToId, sleep } from '../../helpers'
import { useNavState } from '../../hooks'

const props = defineProps<{
  id: string
  item: {
    id: string
    title: string
    select?: () => void
    link?: string
    icon?: {
      src: string
    }
    httpVerb?: string
    deprecated?: boolean
  }
  isActive?: boolean
  hasChildren?: boolean
  open?: boolean
}>()

const emit = defineEmits<{
  (e: 'toggleOpen'): void
}>()

const { hash, isIntersectionEnabled, pathRouting } = useNavState()

// We disable intersection observer on click
const handleClick = async () => {
  if (props.hasChildren) emit('toggleOpen')
  props.item?.select?.()
}

// Build relative URL and add hash
const generateLink = () => {
  if (pathRouting.value) {
    return pathRouting.value.basePath + '/' + props.item.id
  } else {
    const newUrl = new URL(window.location.href)
    newUrl.hash = props.item.id
    return `${newUrl.pathname}${newUrl.search}${newUrl.hash}`
  }
}

// For path routing we want to handle the clicks
const onAnchorClick = async (ev: Event) => {
  if (pathRouting.value) {
    ev.preventDefault()

    // Due to the prevent default
    if (props.hasChildren) emit('toggleOpen')
    props.item?.select?.()

    // Make sure to open the section
    emit('toggleOpen')

    // Disable intersection observer before we scroll
    isIntersectionEnabled.value = false

    // Manually update "hash"
    hash.value = props.item.id

    const url = new URL(window.location.href)
    url.pathname = pathRouting.value.basePath + '/' + props.item.id

    window.history.pushState({}, '', url)
    scrollToId(props.item.id)

    await sleep(100)
    isIntersectionEnabled.value = true
  }
}
</script>
<template>
  <li
    :id="id"
    class="sidebar-group-item">
    <div
      class="sidebar-heading"
      :class="{
        'sidebar-group-item__folder': hasChildren,
        'active_page': isActive,
        'deprecated': item.deprecated ?? false,
      }"
      @click="handleClick">
      <!-- If children are detected then show the nesting icon -->
      <!-- Use &hairsp; to vertically center scalar icon button to the first line of text in the sidebar heading link -->
      <p
        v-if="hasChildren"
        class="sidebar-heading-chevron">
        <ScalarIconButton
          class="toggle-nested-icon"
          :icon="open ? 'ChevronDown' : 'ChevronRight'"
          label="Toggle group"
          size="sm"
          @click.stop="handleClick" />
        &hairsp;
      </p>
      <a
        class="sidebar-heading-link"
        :href="generateLink()"
        @click="onAnchorClick">
        <ScalarIcon
          v-if="item?.icon?.src"
          class="sidebar-icon"
          :icon="item.icon.src as Icon" />
        <p class="sidebar-heading-link-title">
          {{ item.title }}
        </p>
        <p
          v-if="item.httpVerb"
          class="sidebar-heading-link-method">
          &hairsp;
          <HttpMethod
            class="sidebar-heading-type"
            :method="item.httpVerb"
            property="--method-color"
            short />
        </p>
      </a>
    </div>
    <slot v-if="open" />
    <div
      v-if="$slots['action-menu']"
      class="action-menu">
      <slot name="action-menu"></slot>
    </div>
  </li>
</template>
<style scoped>
.sidebar-heading {
  display: flex;
  gap: 6px;

  /* prettier-ignore */
  color: var(--sidebar-color-2, var(--default-theme-color-2, var(--theme-color-2, var(--default-theme-color-2))));
  font-size: var(--theme-mini, var(--default-theme-mini));
  font-weight: var(--theme-semibold, var(--default-theme-semibold));
  word-break: break-word;
  line-height: 1.385;
  max-width: 100%;
  position: relative;
  cursor: pointer;
  border-radius: var(--theme-radius, var(--default-theme-radius));
  flex: 1;
  padding-right: 9px;
  user-select: none;
}
.sidebar-heading.deprecated .sidebar-heading-link-title {
  text-decoration: line-through;
}
.sidebar-heading:hover {
  /* prettier-ignore */
  background: var(--sidebar-item-hover-background, var(--default-sidebar-item-hover-background, var(--theme-background-2, var(--default-theme-background-2))));
}
.sidebar-heading:hover .sidebar-heading-link-title {
  color: var(
    --sidebar-item-hover-color,
    var(--default-sidebar-item-hover-color, currentColor)
  );
}

.active_page.sidebar-heading:hover,
.active_page.sidebar-heading {
  /* prettier-ignore */
  color: var(--sidebar-color-active, var(--default-sidebar-color-active, var(--theme-color-accent, var(--default-theme-color-accent))));
  /* prettier-ignore */
  background: var(--sidebar-item-active-background, var(--default-sidebar-item-active-background, var(--theme-background-accent, var(--default-theme-background-accent))));
}
.active_page.sidebar-heading:hover .sidebar-heading-link-title {
  color: var(
    --sidebar-color-active,
    var(
      --default-sidebar-color-active,
      var(--theme-color-accent, var(--default-theme-color-accent))
    )
  );
}
.sidebar-indent-nested .sidebar-indent-nested .sidebar-heading:before {
  content: '';
  position: absolute;
  top: 0;
  left: calc((var(--sidebar-level, var(--default-sidebar-level)) * 12px));
  width: 1px;
  height: 100%;
  background: var(
    --sidebar-indent-border,
    var(--default-sidebar-indent-border)
  );
}
.sidebar-indent-nested .sidebar-indent-nested .sidebar-heading:hover:before {
  background: var(
    --sidebar-indent-border-hover,
    var(--default-sidebar-indent-border-hover)
  );
}
.sidebar-indent-nested
  .sidebar-indent-nested
  .active_page.sidebar-heading:before {
  background: var(
    --sidebar-indent-border-active,
    var(--default-sidebar-indent-border-active)
  );
}

.sidebar-heading-link {
  text-decoration: none;
  color: inherit;
  padding-right: 12px;
  padding: 6px 0;
  display: flex;
  flex: 1;
  justify-content: space-between;
  gap: 2px;
}
.sidebar-heading p {
  height: fit-content;
  display: flex;
  align-items: center;
}
.sidebar-heading p:empty {
  display: none;
}
/* Sidebar link icon */
.link-icon {
  position: relative;
  left: 4px;
}

.sidebar-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 6px;
  width: 13px;
  height: 13px;
}

.sidebar-icon > svg {
  width: 13px;
  height: 13px;
}

.sidebar-group-item {
  position: relative;
}

/* Folder/page collapse icon */
/* awkward pixel value to deal with hairspace alignment across browser*/
.sidebar-heading-chevron {
  margin: 5px -5.5px 5px -9px;
}
.toggle-nested-icon {
  border: none;
  color: currentColor;
  padding: 3px;
  color: var(--sidebar-color-2, var(--default-sidebar-color-2));
}
.active_page .toggle-nested-icon {
  color: var(
    --sidebar-color-active,
    var(
      --default-sidebar-color-active,
      var(--theme-color-accent, var(--default-theme-color-accent))
    )
  );
}

.toggle-nested-icon:hover,
.toggle-nested-icon:focus-visible {
  color: currentColor;
}

.action-menu {
  position: absolute;
  top: 5px;
  right: 5px;
  display: flex;
  gap: 6px;
}
/**
* Some awkwardness to make the dropdown buttons hidden when not hovered
* but still show when the panel is open and focused
*/
.action-menu :deep(.button-wrapper button) {
  /* Hide the icons by default */
  opacity: 0;
  width: 20px;
  height: 20px;
  padding: 4px;
}
.action-menu:hover :deep(.button-wrapper button),
.action-menu :deep(.button-wrapper button:hover),
.sidebar-heading:hover ~ .action-menu :deep(.button-wrapper button),
.action-menu :deep(.button-wrapper button[aria-expanded='true']) {
  opacity: 1;
}
.sidebar-heading:has(~ .action-menu:hover) {
  /* prettier-ignore */
  color: var(--sidebar-color-1, var(--default-sidebar-color-1), var(--theme-color-1, var(--default-theme-color-1)));
  /* prettier-ignore */
  background: var(--sidebar-item-hover-background, var(--default-sidebar-item-hover-background), var(--theme-background-2, var(--default-theme-background-2)));
}

.sidebar-heading-type {
  min-width: 3.9em;
  overflow: hidden;
  border-radius: 30px;
  padding: 0 3px;
  line-height: 14px;
  flex-shrink: 0;
  color: white;
  color: color-mix(
    in srgb,
    var(--method-color, var(--theme-color-1)),
    transparent 0%
  );
  background: var(--method-color, var(--theme-background-3));
  background: color-mix(
    in srgb,
    var(--method-color, var(--theme-background-3)),
    transparent 90%
  );
  text-transform: uppercase;
  font-size: 8.5px;
  font-weight: bold;
  text-align: center;
  position: relative;
  font-family: var(--theme-font-code, var(--default-theme-font-code));
  white-space: nowrap;
  margin-left: 3px;
}
.active_page .sidebar-heading-type {
  background: transparent;
}
.active_page .sidebar-heading-type {
  background: var(--method-color);
  color: color-mix(in srgb, var(--method-color), white 85%);
}
.dark-mode .active_page .sidebar-heading-type {
  background: var(--method-color);
  color: color-mix(in srgb, var(--method-color), black 80%);
}
.sidebar-group-item__folder {
  color: var(
    --sidebar-color-1,
    var(
      --default-sidebar-color-1,
      var(--theme-color-1, var(--default-theme-color-1))
    )
  );
}
.sidebar-group-item__folder .sidebar-heading-type {
  display: none;
}
</style>
