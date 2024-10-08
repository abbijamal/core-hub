<script setup lang="ts">
import mediumZoom from 'medium-zoom'
import { withoutTrailingSlash, joinURL } from 'ufo'
import type { BlogPost } from '~/types'

definePageMeta({
  primary: 'green',
  heroBackground: 'opacity-30'
})

const route = useRoute()
const { copy } = useCopyToClipboard()
const { toc } = useAppConfig()
const { url } = useSiteConfig()

const { data: post } = await useAsyncData(route.path, () => queryContent<BlogPost>(route.path).findOne())
if (!post.value) {
  throw createError({ statusCode: 404, statusMessage: 'Post not found', fatal: true })
}

const { data: surround } = await useAsyncData(`${route.path}-surround`, () => queryContent()
  .where({ _extension: 'md', navigation: { $ne: false }, _path: { $regex: /^\/blog/ } })
  .without(['body', 'excerpt'])
  .sort({ date: -1 })
  .findSurround(withoutTrailingSlash(route.path))
)

const title = post.value.head?.title || post.value.title
const description = post.value.head?.description || post.value.description

useSeoMeta({
  titleTemplate: '%s · NuxtHub Blog',
  title,
  description,
  ogDescription: description,
  ogTitle: `${title} · NuxtHub Blog`
})

if (post.value.image) {
  defineOgImage({ url: joinURL(url, post.value.image) })
}

const socialLinks = computed(() => [{
  icon: 'i-simple-icons-linkedin',
  to: `https://www.linkedin.com/sharing/share-offsite/?url=https://hub.nuxt.com${post.value._path}`
}, {
  icon: 'i-simple-icons-twitter',
  to: `https://twitter.com/intent/tweet?text=${encodeURIComponent(`${post.value.title}\n\n`)}https://hub.nuxt.com${post.value._path}`
}])

const links = computed(() => [{
  icon: 'i-heroicons-pencil-square',
  label: 'Edit this post',
  to: `${toc.bottom.edit}/${post?.value?._file}`,
  target: '_blank'
}, ...asideLinks].filter(Boolean))

function copyLink() {
  copy(`https://hub.nuxt.com${post.value._path}`, { title: 'Post URL to clipboard' })
}
onMounted(() => {
  mediumZoom('[data-zoom-src]', {
    margin: 5
  })
})
</script>

<template>
  <UContainer>
    <UPage>
      <UPageHeader :title="post.title" :description="post.description" :ui="{ headline: 'flex flex-col gap-y-8 items-start', description: 'text-gray-700 dark:text-gray-300' }">
        <template #headline>
          <UBreadcrumb :links="[{ label: 'Blog', icon: 'i-ph-newspaper-duotone', to: '/blog' }, { label: post.title }]" :ui="{ wrapper: 'max-w-full' }" />
          <div class="flex items-center space-x-2">
            <span>
              {{ post.badge?.label || 'Article' }}
            </span>
            <span class="text-gray-500 dark:text-gray-400">&middot;&nbsp;&nbsp;<time>{{ formatDateByLocale('en', post.date) }}</time></span>
          </div>
        </template>
        <div class="flex flex-wrap items-center gap-3 mt-4">
          <div class="mt-4 flex flex-wrap items-center gap-6">
            <UButton
              v-for="author in post.authors" :key="author.username" :to="author.to" target="_blank"
              color="white" variant="ghost" class="-my-1.5 -mx-2.5"
            >
              <UAvatar :src="author.avatar?.src" :alt="author.name" />

              <div class="text-left">
                <p class="font-medium">
                  {{ author.name }}
                </p>
                <p class="text-gray-500 dark:text-gray-400 leading-4">
                  {{ `@${author.username}` }}
                </p>
              </div>
            </UButton>
          </div>
        </div>
      </UPageHeader>

      <UPage>
        <UPageBody prose class="dark:text-gray-300 dark:prose-pre:!bg-gray-800/60">
          <ContentRenderer v-if="post && post.body" :value="post" />

          <div class="flex items-center justify-between mt-12 not-prose">
            <UButton to="/blog" variant="link" :padded="false">
              ← Back to blog
            </UButton>
            <div class="flex justify-end items-center gap-1.5">
              <UButton icon="i-ph-link-simple" v-bind="($ui.button.secondary as any)" @click="copyLink">
                Copy URL
              </UButton>
              <UButton
                v-for="(link, index) in socialLinks"
                :key="index"
                v-bind="link"
                variant="ghost"
                color="gray"
                target="_blank"
              />
            </div>
          </div>

          <hr v-if="surround?.length">

          <UContentSurround :surround="surround" />
        </UPageBody>

        <template #right>
          <UContentToc v-if="post.body && post.body.toc" :links="post.body.toc.links">
            <template #bottom>
              <div class="hidden lg:block space-y-6">
                <UPageLinks title="Links" :links="links" />
              </div>
            </template>
          </UContentToc>
        </template>
      </UPage>
    </UPage>
  </UContainer>
</template>
