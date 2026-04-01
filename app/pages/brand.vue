<template>
  <div class="flex w-full gap-8 space-y-8 px-4 py-32">
    <div class="mx-auto max-w-3xl flex-1">
      <div class="grid grid-cols-2 gap-2 rounded-md bg-muted p-1">
        <button
          v-for="tab in tabs" :key="tab.id"
          class="w-full rounded-sm px-3 py-2 text-sm font-medium whitespace-nowrap transition-colors"
          :class="activeTab === tab.id ? 'bg-background text-foreground' : 'text-muted-foreground hover:text-foreground'"
          @click="activeTab = tab.id"
        >
          {{ tab.label }}
        </button>
      </div>

      <div v-if="activeTab === 'logos'" class="space-y-8 rounded-lg border p-4 md:p-6">
        <img :src="selectedWordmark.image" :alt="selectedWordmark.name" class="mx-auto h-auto w-full object-contain p-12" :class="selectedWordmark.bgClass">

        <div class="flex w-full items-center justify-between">
          <span class="text-sm font-medium">{{ selectedWordmark.name }}</span>
          <button :aria-label="`Download ${selectedWordmark.name}`" class="transition-transform hover:scale-110" @click="handleDownloadImage(selectedWordmark, selectedWordmarkIndex, wordmarkActions)">
            <icon :name="wordmarkActions[selectedWordmarkIndex]!.icon.value" size="20" />
          </button>
        </div>

        <div class="flex items-center justify-center gap-2">
          <button
            v-for="(asset, index) in WORDMARKS" :key="asset.name"
            :aria-label="`Show ${asset.name}`" class="size-3 rounded-full transition-all hover:ring-2 hover:ring-primary/50"
            :class="index === selectedWordmarkIndex ? 'ring-2 ring-primary' : 'ring-1 ring-muted'" @click="selectedWordmarkIndex = index"
          />
        </div>

        <div class="grid w-full grid-cols-3 gap-4">
          <div v-for="(asset, index) in SYMBOLS" :key="asset.name" class="min-w-0 space-y-2">
            <img :src="asset.image" :alt="asset.name" class="mx-auto block size-32 object-contain">

            <div class="flex w-full items-center justify-between gap-2">
              <span class="truncate text-sm font-medium">{{ asset.name }}</span>
              <button :aria-label="`Download ${asset.name}`" class="shrink-0 transition-transform hover:scale-110" @click="handleDownloadImage(asset, index, symbolActions)">
                <icon :name="symbolActions[index]!.icon.value" size="20" />
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="activeTab === 'colors'" class="space-y-8 rounded-lg border p-4">
        <div v-for="category in COLOR_CATEGORIES" :id="category.id" :key="category.name" class="space-y-4">
          <h3 class="text-sm font-semibold">
            {{ category.name }}
          </h3>

          <div class="grid w-full gap-4" :class="category.id === 'brand' ? 'grid-cols-2' : 'grid-cols-3'">
            <div v-for="(color, index) in category.colors" :key="color.name" class="mx-auto flex flex-col items-center gap-2" :class="category.id === 'brand' ? 'w-36 md:w-44' : 'w-24 md:w-32'">
              <div class="aspect-square w-full" :style="{ backgroundColor: `var(${color.var})` }" />

              <div class="flex w-full flex-col">
                <div class="flex w-full items-center justify-between">
                  <span class="truncate text-sm font-medium">{{ color.name }}</span>
                  <button :aria-label="`Copy ${color.name}`" class="transition-transform hover:scale-110" @click="handleCopyColor(color.var, index, category.actions)">
                    <icon :name="category.actions[index]!.icon.value" size="20" />
                  </button>
                </div>

                <span class="text-xs text-muted-foreground">{{ colorValues[color.var] }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const { public: { baseURL } } = useRuntimeConfig()
const { createActionHandler } = useActionIcon()
const { colorMode } = useTheme()
const colorValues = ref<Record<string, string>>({})
const tabs = [{ id: "logos", label: "Wordmarks & Symbols" }, { id: "colors", label: "Colors" }] as const
const activeTab = ref<typeof tabs[number]["id"]>("logos")
const selectedWordmarkIndex = ref(0)
const symbolActions = SYMBOLS.map(() => createActionHandler("mdi:download"))
const wordmarkActions = WORDMARKS.map(() => createActionHandler("mdi:download"))
const selectedWordmark = computed(() => WORDMARKS[selectedWordmarkIndex.value] ?? WORDMARKS[0]!)

const COLOR_CATEGORIES = [
  { id: "brand", name: "Brand Colors", colors: BRAND_COLORS, actions: BRAND_COLORS.map(() => createActionHandler("mdi:content-copy")) },
  { id: "neutral", name: "Neutral Colors", colors: NEUTRAL_COLORS, actions: NEUTRAL_COLORS.map(() => createActionHandler("mdi:content-copy")) },
  { id: "status", name: "Status Colors", colors: STATUS_COLORS, actions: STATUS_COLORS.map(() => createActionHandler("mdi:content-copy")) },
]

function handleDownloadImage(logo: { name: string, image: string }, index: number, actions: ReturnType<typeof createActionHandler>[]) {
  if (!actions[index]) {
    return
  }

  const link = document.createElement("a")
  link.href = logo.image
  link.download = logo.name.replace(/\s+/g, "-").toLowerCase()
  link.click()
  actions[index].triggerSuccess()
}

async function handleCopyColor(colorVar: string, index: number, actions: ReturnType<typeof createActionHandler>[]) {
  if (!actions[index]) {
    return
  }

  const value = colorValues.value[colorVar]
  if (!value || value === "—") {
    return
  }

  await actions[index].triggerCopy(value)
}

function syncColorValues() {
  for (const color of COLOR_CATEGORIES.flatMap(category => category.colors)) {
    colorValues.value[color.var] = getComputedStyle(document.documentElement).getPropertyValue(color.var).trim() || "—"
  }
}

onMounted(syncColorValues)

watch(colorMode, () => {
  syncColorValues()
}, { flush: "post" })

useHead({
  title: "Brand Resources",
  link: [{ rel: "canonical", href: `${baseURL}/brand` }],
  meta: [{ name: "description", content: "LinKiosk brand resources and guidelines." }],
})
</script>
