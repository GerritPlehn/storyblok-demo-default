<script setup>
const props = defineProps({
  logo: Object,
  disable_transparency: Boolean,
  auto_nav: Boolean,
  auto_nav_folder: String,
  nav: Object,
  buttons: Object,
  light: Boolean,
})

const folderStories = ref(null)

const getFolderStories = async () => {
  const storyblokApi = useStoryblokApi()
  const { data } = await storyblokApi.get('cdn/stories', {
    version: 'draft',
    level: props.auto_nav_folder ? 2 : 1,
    excluding_slugs: 'site-config,error-404',
    excluding_fields: 'body',
    starts_with: props.auto_nav_folder,
    per_page: 5,
  })
  folderStories.value = !props.auto_nav_folder
    ? data.stories.filter(
        (story) => story.parent_id === 0 || story.parent_id === null
      )
    : data.stories
}

getFolderStories()

watch(
  () => props.auto_nav_folder,
  () => getFolderStories()
)

const mobileNavOpen = ref(false)

const toggleMobileNav = () => {
  mobileNavOpen.value = !mobileNavOpen.value
}

const route = useRoute()
watch(route, () => {
  mobileNavOpen.value = false
})

const headerClasses = ref('h-32')
const logoScale = ref('scale-100')

const headerBg = computed(() => {
  return props.light ? 'bg-white' : 'bg-neutral-900'
})

const headerTransparency = computed(() => {
  return props.disable_transparency ? '' : 'bg-opacity-80 backdrop-blur-lg'
})

onMounted(() => {
  window.addEventListener('scroll', () => {
    if (window.scrollY > 60) {
      headerClasses.value = ' shadow-md h-20'
      logoScale.value = 'scale-75'
    } else {
      headerClasses.value = 'h-32'
      logoScale.value = 'scale-100'
    }
  })
})
</script>

<template>
  <header
    class="fixed left-0 top-0 z-[99] w-full transition-all duration-700"
    :class="[headerClasses, headerBg, headerTransparency]"
  >
    <div
      class="mx-auto flex h-full w-full max-w-[1536px] items-center justify-between px-4 lg:justify-start lg:px-8"
    >
      <NuxtLink to="/" class="flex shrink-0">
        <img
          :src="logo.filename"
          :alt="logo.alt"
          class="pointer-events-none max-h-[80px] w-full max-w-[180px] origin-left object-contain transition-transform duration-700 xl:max-w-[250px]"
          :class="logoScale"
        />
      </NuxtLink>
      <nav class="main-nav invisible mx-auto hidden lg:visible lg:block">
        <ul v-if="!auto_nav">
          <li v-for="item in nav" :key="item._uid">
            <NavItem
              class="hover:underline hover:underline-offset-2"
              :class="light ? 'text-dark' : 'text-white'"
              :item="item"
            />
          </li>
        </ul>
        <ul v-else>
          <li v-for="story in folderStories" :key="story.uuid">
            <NuxtLink
              :to="story.full_slug"
              class="cursor-pointer transition-colors hover:underline hover:underline-offset-2"
              :class="light ? 'text-dark' : 'text-white'"
            >
              {{ story.name }}
            </NuxtLink>
          </li>
        </ul>
      </nav>
      <nav
        class="invisible ml-auto hidden md:visible md:mr-8 md:block lg:ml-0 lg:mr-0"
      >
        <ul class="flex items-center space-x-4 xl:space-x-8">
          <li v-for="button in buttons" :key="button._uid">
            <Button :button="button" />
          </li>
        </ul>
      </nav>
      <MobileNavToggle
        @click="toggleMobileNav"
        :color="light ? 'bg-dark' : 'bg-light'"
      />
    </div>
  </header>
  <MobileNav
    :mobileNavOpen="mobileNavOpen"
    :headerNav="nav"
    :autoNav="auto_nav"
    :folderStories="folderStories"
  />
  <pre></pre>
</template>

<style scoped>
header nav.main-nav a.router-link-active {
  @apply underline decoration-primary decoration-4 underline-offset-4;
}

header nav.main-nav ul {
  @apply flex space-x-4 xl:space-x-8 xl:text-lg;
}
</style>
