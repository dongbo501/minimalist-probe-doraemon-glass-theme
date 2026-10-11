<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useAppStore } from '@/stores/app'

const appStore = useAppStore()

const isLoaded = ref(false)
const hasError = ref(false)

const showBackground = computed(() => appStore.backgroundEnabled)
const currentUrl = computed(() => showBackground.value ? appStore.currentBackgroundUrl : '')
const backgroundType = computed(() => appStore.backgroundType)
const hasCustomBackground = computed(() => showBackground.value && !!currentUrl.value)
const showBackgroundOverlay = computed(() => appStore.backgroundOverlay > 0)

const backgroundStyle = computed(() => {
  const blur = appStore.backgroundBlur
  return {
    filter: blur > 0 ? `blur(${blur}px)` : 'none',
    opacity: appStore.backgroundType === 'video' && !isLoaded.value ? 0 : 1,
  }
})

const backgroundContainerStyle = computed(() => {
  const overlay = appStore.backgroundOverlay
  if (overlay >= 0)
    return {}

  return { opacity: 1 - Math.abs(overlay) / 100 }
})

const overlayStyle = computed(() => {
  const overlay = appStore.backgroundOverlay
  if (overlay <= 0)
    return {}

  return { backgroundColor: `rgba(0, 0, 0, ${overlay / 100})` }
})

const showLoadedBackground = computed(() =>
  hasCustomBackground.value && isLoaded.value && !hasError.value,
)

const showMediaBackground = computed(() =>
  hasCustomBackground.value && !hasError.value && (backgroundType.value === 'video' || showLoadedBackground.value),
)

const showDefaultBackground = computed(() =>
  !hasCustomBackground.value || !showMediaBackground.value || hasError.value,
)

const showLoadingBackground = computed(() =>
  hasCustomBackground.value && backgroundType.value === 'video' && !isLoaded.value && !hasError.value,
)

const showFallbackBackground = computed(() =>
  hasCustomBackground.value && backgroundType.value === 'video' && hasError.value,
)

let imageLoader: HTMLImageElement | null = null

function clearImageLoader() {
  if (imageLoader) {
    imageLoader.onload = null
    imageLoader.onerror = null
    imageLoader = null
  }
}

function loadImage(url: string) {
  isLoaded.value = false
  hasError.value = false

  clearImageLoader()

  imageLoader = new Image()
  imageLoader.onload = () => {
    isLoaded.value = true
    hasError.value = false
  }
  imageLoader.onerror = () => {
    isLoaded.value = false
    hasError.value = true
  }
  imageLoader.src = url
}

const videoRef = ref<HTMLVideoElement | null>(null)

function resetBackgroundState() {
  clearImageLoader()

  if (videoRef.value) {
    videoRef.value.pause()
    videoRef.value.removeAttribute('src')
    videoRef.value.load()
  }

  isLoaded.value = false
  hasError.value = false
}

function handleVideoLoaded() {
  isLoaded.value = true
  hasError.value = false
}
function handleVideoError() {
  isLoaded.value = false
  hasError.value = true
}

watch([showBackground, currentUrl, backgroundType], ([enabled, url, type]) => {
  if (!enabled || !url) {
    resetBackgroundState()
    return
  }

  if (type === 'image') {
    loadImage(url)
  }
  else if (type === 'video') {
    clearImageLoader()
    isLoaded.value = false
    hasError.value = false
  }
}, { immediate: true })

onUnmounted(() => {
  resetBackgroundState()
})
</script>

<template>
  <div class="background-container" :style="backgroundContainerStyle">
    <Transition name="fade">
      <div v-if="showDefaultBackground" class="default-background" aria-hidden="true">
        <!-- 液态玻璃需要有颜色可以折射：天空里铺几团缓慢漂移的柔光 -->
        <span class="lg-glow lg-glow--gold" />
        <span class="lg-glow lg-glow--pink" />
        <span class="lg-glow lg-glow--white" />
        <div class="dora-sun" />
        <div class="dora-sky-pattern" />
        <span class="dora-cloud dora-cloud--1" />
        <span class="dora-cloud dora-cloud--2" />
        <span class="dora-cloud dora-cloud--3" />
        <span class="dora-cloud dora-cloud--4" />
        <!-- 玻璃卡片后面的几朵云：清透玻璃要有景物可看 -->
        <span class="dora-cloud dora-cloud--5" />
        <span class="dora-cloud dora-cloud--6" />
        <span class="dora-cloud dora-cloud--7" />
        <span class="dora-flyer"><i /></span>

        <!-- 大雄家附近的小镇：房子、电线杆、空地上的三根水泥管 -->
        <svg class="dora-town" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <pattern id="dora-town-tile" width="1200" height="220" patternUnits="userSpaceOnUse">
              <path class="t-far" d="M0 204V150H60V130H110V160H170V120H210V140H300V165H380V110H420V100H440V110H470V150H560V135H640V160H720V125H780V145H880V115H930V150H1010V130H1090V155H1150V140H1200V204Z" />

              <g class="t-wires">
                <path d="M-350 74Q-50 112 250 74M250 74Q550 112 850 74M850 74Q1150 112 1450 74" />
                <path d="M-350 84Q-50 120 250 84M250 84Q550 120 850 84M850 84Q1150 120 1450 84" />
              </g>

              <!-- 两层小楼 -->
              <rect class="t-wall" x="40" y="148" width="140" height="56" />
              <path class="t-roof" d="M30 152H190L176 138H44Z" />
              <rect class="t-wall" x="62" y="104" width="96" height="36" />
              <path class="t-roof" d="M50 108L110 78L170 108Z" />
              <rect class="t-win t-lit" x="76" y="114" width="20" height="14" rx="1.5" />
              <rect class="t-win" x="124" y="114" width="20" height="14" rx="1.5" />
              <rect class="t-win" x="54" y="162" width="26" height="18" rx="1.5" />
              <rect class="t-win t-lit" x="94" y="162" width="26" height="18" rx="1.5" />
              <rect class="t-door" x="140" y="166" width="22" height="38" rx="1.5" />
              <rect class="t-fence" x="18" y="182" width="206" height="22" />
              <path class="t-fence-line" d="M18 189H224M18 196H224M52 182V204M86 182V204M120 182V204M154 182V204M188 182V204" />

              <!-- 电线杆 -->
              <rect class="t-pole" x="247" y="60" width="6" height="144" />
              <rect class="t-pole" x="232" y="72" width="36" height="4" rx="1" />
              <rect class="t-pole" x="847" y="60" width="6" height="144" />
              <rect class="t-pole" x="832" y="72" width="36" height="4" rx="1" />

              <!-- 空地和三根水泥管 -->
              <path class="t-grass" d="M284 204l6-10 4 10M318 204l5-8 5 8M470 204l6-11 5 11M520 204l4-8 5 8M540 204l6-10 4 10" />
              <circle class="t-pipe" cx="362" cy="178" r="26" />
              <circle class="t-pipe" cx="416" cy="178" r="26" />
              <circle class="t-pipe" cx="389" cy="132" r="26" />
              <circle class="t-hole" cx="362" cy="178" r="17" />
              <circle class="t-hole" cx="416" cy="178" r="17" />
              <circle class="t-hole" cx="389" cy="132" r="17" />
              <path class="t-pipe-rim" d="M346 166a20 20 0 0 1 12-8M400 166a20 20 0 0 1 12-8M373 120a20 20 0 0 1 12-8" />

              <!-- 带阳台的房子 -->
              <rect class="t-wall" x="590" y="120" width="190" height="84" />
              <path class="t-roof" d="M576 124L685 76L794 124Z" />
              <rect class="t-win t-lit" x="610" y="134" width="24" height="16" rx="1.5" />
              <rect class="t-win" x="732" y="134" width="24" height="16" rx="1.5" />
              <rect class="t-win t-lit" x="702" y="166" width="30" height="20" rx="1.5" />
              <rect class="t-door" x="626" y="168" width="22" height="36" rx="1.5" />
              <path class="t-rail" d="M606 154H702M606 154V164M630 154V164M654 154V164M678 154V164M702 154V164M606 164H702" />

              <!-- 树 -->
              <rect class="t-trunk" x="801" y="160" width="8" height="44" rx="2" />
              <circle class="t-tree" cx="805" cy="140" r="26" />
              <circle class="t-tree" cx="788" cy="154" r="18" />
              <circle class="t-tree" cx="824" cy="154" r="18" />

              <!-- 公寓 -->
              <rect class="t-wall" x="890" y="110" width="160" height="94" />
              <rect class="t-roof" x="884" y="104" width="172" height="8" rx="2" />
              <rect class="t-roof" x="1010" y="86" width="24" height="18" rx="2" />
              <rect class="t-win" x="906" y="126" width="30" height="18" rx="1.5" />
              <rect class="t-win t-lit" x="956" y="126" width="30" height="18" rx="1.5" />
              <rect class="t-win" x="1006" y="126" width="30" height="18" rx="1.5" />
              <rect class="t-win t-lit" x="906" y="160" width="30" height="18" rx="1.5" />
              <rect class="t-win" x="956" y="160" width="30" height="18" rx="1.5" />
              <rect class="t-win t-lit" x="1006" y="160" width="30" height="18" rx="1.5" />
              <rect class="t-fence" x="1068" y="184" width="132" height="20" />
              <path class="t-fence-line" d="M1068 191H1200M1068 197H1200M1100 184V204M1134 184V204M1168 184V204" />

              <rect class="t-ground" x="0" y="204" width="1200" height="16" />
              <path class="t-road" d="M20 213H80M160 213H220M300 213H360M440 213H500M580 213H640M720 213H780M860 213H920M1000 213H1060M1140 213H1200" />
            </pattern>
          </defs>
          <rect width="100%" height="100%" fill="url(#dora-town-tile)" />
        </svg>

        <!-- 水泥管旁边的空地上：大雄和静香在玩抛球。坐标和小镇图案的第一块（1200px）对齐 -->
        <svg class="dora-kids" width="1200" height="220" viewBox="0 0 1200 220" xmlns="http://www.w3.org/2000/svg">
          <g transform="translate(518 204) scale(0.68) translate(-518 -204)">
            <!-- 大雄：黄上衣、藏青短裤、圆眼镜，面朝右 -->
            <g class="kid kid--nobita">
              <ellipse class="k-shoe-n" cx="472" cy="203" rx="4.5" ry="2.2" />
              <ellipse class="k-shoe-n" cx="485" cy="203" rx="4.5" ry="2.2" />
              <rect class="k-skin" x="470" y="189" width="4" height="13" />
              <rect class="k-skin" x="483" y="189" width="4" height="13" />
              <rect class="k-shorts" x="467" y="181" width="23" height="10" rx="2" />
              <g class="kid__arm kid__arm--back">
                <rect class="k-shirt" x="463" y="166" width="5" height="10" rx="2.5" />
                <circle class="k-skin" cx="465.5" cy="178" r="2.6" />
              </g>
              <rect class="k-shirt" x="466" y="163" width="25" height="21" rx="6" />
              <g class="kid__arm kid__arm--front">
                <rect class="k-shirt" x="488" y="166" width="5" height="10" rx="2.5" />
                <circle class="k-skin" cx="490.5" cy="178" r="2.6" />
              </g>
              <circle class="k-skin" cx="478" cy="151" r="11" />
              <path class="k-hair" d="M467 151q-1-13 11-13t11 13q-2-6-6-7q-2 2-5 1q-3 2-6 0q-4 2-5 6z" />
              <g class="k-glasses">
                <circle cx="474.5" cy="152.5" r="3.4" />
                <circle cx="482.5" cy="152.5" r="3.4" />
                <path d="M477.9 152.5h1.2" />
              </g>
              <circle class="k-eye" cx="475" cy="152.8" r="1" />
              <circle class="k-eye" cx="483" cy="152.8" r="1" />
              <path class="k-mouth" d="M476 158q2 1.8 4.4 0" />
            </g>

            <!-- 静香：粉色连衣裙、两根小辫子，面朝左 -->
            <g class="kid kid--shizuka">
              <ellipse class="k-shoe-s" cx="552" cy="203" rx="4.5" ry="2.2" />
              <ellipse class="k-shoe-s" cx="564" cy="203" rx="4.5" ry="2.2" />
              <rect class="k-skin" x="550" y="189" width="4" height="13" />
              <rect class="k-skin" x="562" y="189" width="4" height="13" />
              <g class="kid__arm kid__arm--back">
                <rect class="k-dress" x="569" y="166" width="5" height="10" rx="2.5" />
                <circle class="k-skin" cx="571.5" cy="178" r="2.6" />
              </g>
              <path class="k-dress" d="M550 163h16q4 0 5 4l3 23h-32l3-23q1-4 5-4z" />
              <g class="kid__arm kid__arm--front">
                <rect class="k-dress" x="542" y="166" width="5" height="10" rx="2.5" />
                <circle class="k-skin" cx="544.5" cy="178" r="2.6" />
              </g>
              <g class="kid__tail kid__tail--left">
                <ellipse class="k-hair" cx="545.5" cy="158" rx="3.2" ry="5.5" />
                <circle class="k-tie" cx="546" cy="152.6" r="1.7" />
              </g>
              <g class="kid__tail kid__tail--right">
                <ellipse class="k-hair" cx="570.5" cy="158" rx="3.2" ry="5.5" />
                <circle class="k-tie" cx="570" cy="152.6" r="1.7" />
              </g>
              <circle class="k-skin" cx="558" cy="151" r="11" />
              <path class="k-hair" d="M547 151q-1-13 11-13t11 13q-2-5-5-6q-3 3-6 1q-3 2-6 0q-4 1-5 5z" />
              <circle class="k-eye" cx="554.5" cy="152.5" r="1.2" />
              <circle class="k-eye" cx="561.5" cy="152.5" r="1.2" />
              <circle class="k-blush" cx="552" cy="156" r="1.8" />
              <circle class="k-blush" cx="564" cy="156" r="1.8" />
              <path class="k-mouth" d="M556 157.8q2 1.8 4 0" />
            </g>

            <!-- 球：横向匀速，纵向每一程画一道抛物线 -->
            <g class="kid-ball">
              <g class="kid-ball__arc">
                <circle class="k-ball" cx="494" cy="168" r="5" />
                <path class="k-ball-stripe" d="M489.2 166.6q4.8 3 9.6 0" />
              </g>
            </g>
          </g>
        </svg>
      </div>
    </Transition>
    <Transition name="fade">
      <div v-if="showLoadingBackground" class="background-loading" />
    </Transition>
    <Transition name="fade">
      <div v-if="showFallbackBackground" class="background-loading" />
    </Transition>
    <Transition name="fade">
      <div v-if="showMediaBackground" class="background-media" :style="backgroundStyle">
        <div
          v-if="backgroundType === 'image'"
          class="background-image"
          :style="{ backgroundImage: `url(${currentUrl})` }"
        />
        <video
          v-else-if="backgroundType === 'video'"
          ref="videoRef"
          class="background-video"
          :src="currentUrl ?? undefined"
          autoplay
          loop
          muted
          preload="auto"
          playsinline
          @loadeddata="handleVideoLoaded"
          @canplay="handleVideoLoaded"
          @error="handleVideoError"
        />
      </div>
    </Transition>
    <div v-if="showBackgroundOverlay" class="background-overlay" :style="overlayStyle" />
  </div>
</template>

<style scoped>
.background-container {
  position: fixed;
  inset: 0;
  z-index: -1;
  overflow: hidden;
}

.default-background {
  position: absolute;
  inset: 0;
  overflow: hidden;
  /* 晴空：上浅下深的哆啦A梦蓝，地平线附近有一层暖白的光 */
  background:
    radial-gradient(ellipse 80% 38% at 50% 100%, rgb(255 255 255 / 0.34), transparent 70%),
    linear-gradient(180deg, #5cc8f7 0%, #1aa6ec 34%, #0096e0 62%, #0a86cf 100%);
}

.dark .default-background {
  /* 夜空：大雄的小镇睡着了，哆啦A梦在壁橱里待机充电 */
  background:
    radial-gradient(ellipse 70% 40% at 50% 100%, rgb(0 150 224 / 0.22), transparent 70%),
    linear-gradient(180deg, #020d1c 0%, #04213b 46%, #073357 100%);
}

.lg-glow,
.dora-sun,
.dora-sky-pattern,
.dora-cloud,
.dora-flyer,
.dora-town,
.dora-kids {
  position: absolute;
  pointer-events: none;
}

/* 柔光：铃铛金、任意门粉和一团白光，玻璃卡片从它们上面滑过时会带出淡淡的色彩 */
.lg-glow {
  display: block;
  aspect-ratio: 1;
  border-radius: 50%;
  animation: lg-glow-drift 36s ease-in-out infinite alternate;
}

.lg-glow--gold {
  top: 4%;
  right: 8%;
  width: min(34rem, 70vw);
  background: radial-gradient(circle, rgb(255 215 0 / 0.6), rgb(255 215 0 / 0) 66%);
  /* 金光和粉光各走一条游走路线，同时一呼一吸地明暗、胀缩，两者节奏错开 */
  animation:
    lg-glow-wander-gold 24s ease-in-out infinite,
    lg-glow-breathe 6s ease-in-out infinite alternate;
}

.lg-glow--pink {
  top: 44%;
  left: 12%;
  width: min(38rem, 80vw);
  background: radial-gradient(circle, rgb(242 124 168 / 0.5), rgb(242 124 168 / 0) 66%);
  animation:
    lg-glow-wander-pink 30s ease-in-out infinite,
    lg-glow-breathe 7.5s ease-in-out -3s infinite alternate;
}

/* 光团里有一颗更亮的光心在绕圈，像铃铛和任意门上流动的反光 */
.lg-glow--gold::before,
.lg-glow--pink::before {
  content: '';
  position: absolute;
  inset: 16%;
  border-radius: 50%;
  animation: lg-glow-orbit 14s linear infinite;
}

.lg-glow--gold::before {
  background: radial-gradient(circle at 30% 30%, rgb(255 240 140 / 0.6), rgb(255 215 0 / 0) 42%);
}

.lg-glow--pink::before {
  background: radial-gradient(circle at 30% 30%, rgb(255 120 150 / 0.5), rgb(242 124 168 / 0) 42%);
  animation-duration: 18s;
  animation-direction: reverse;
}

.lg-glow--white {
  top: 22%;
  left: 40%;
  width: min(30rem, 66vw);
  background: radial-gradient(circle, rgb(255 255 255 / 0.38), rgb(255 255 255 / 0) 66%);
  animation-duration: 52s;
}

.dark .lg-glow--gold {
  background: radial-gradient(circle, rgb(255 215 0 / 0.12), rgb(255 215 0 / 0) 66%);
}

.dark .lg-glow--pink {
  background: radial-gradient(circle, rgb(139 92 246 / 0.3), rgb(139 92 246 / 0) 66%);
}

.dark .lg-glow--gold::before {
  background: radial-gradient(circle at 30% 30%, rgb(255 225 80 / 0.16), rgb(255 215 0 / 0) 42%);
}

.dark .lg-glow--pink::before {
  background: radial-gradient(circle at 30% 30%, rgb(167 139 250 / 0.32), rgb(139 92 246 / 0) 42%);
}

.dark .lg-glow--white {
  background: radial-gradient(circle, rgb(0 150 224 / 0.36), rgb(0 150 224 / 0) 66%);
}

/* 太阳：左上角露出半张脸，光芒缓慢旋转 */
.dora-sun {
  top: -15rem;
  left: -13rem;
  width: 32rem;
  height: 32rem;
  border-radius: 50%;
  background:
    radial-gradient(circle, #fff7c2 0 5.5rem, #ffe45c 5.6rem 7.2rem, rgb(255 228 92 / 0.35) 7.3rem, transparent 11rem),
    repeating-conic-gradient(from 0deg, rgb(255 255 255 / 0.22) 0deg 7deg, transparent 7deg 20deg);
  mask-image: radial-gradient(circle, #000 42%, transparent 70%);
  -webkit-mask-image: radial-gradient(circle, #000 42%, transparent 70%);
  animation: dora-sun-spin 120s linear infinite;
}

/* 月亮：夜里换成左上角的一轮金色满月 */
.dark .dora-sun {
  top: -4.5rem;
  left: -4.5rem;
  width: 15rem;
  height: 15rem;
  background:
    radial-gradient(circle at 58% 60%, rgb(214 170 40 / 0.4) 0 1.1rem, transparent 1.15rem),
    radial-gradient(circle at 72% 44%, rgb(214 170 40 / 0.32) 0 0.7rem, transparent 0.75rem),
    radial-gradient(circle at 46% 76%, rgb(214 170 40 / 0.3) 0 0.55rem, transparent 0.6rem),
    radial-gradient(circle at 60% 60%, #fff6c2, #ffd700 68%, #f2c200);
  box-shadow:
    0 0 0 1.2rem rgb(255 215 0 / 0.07),
    0 0 0 2.6rem rgb(255 215 0 / 0.04),
    0 0 5rem rgb(255 215 0 / 0.3);
  mask-image: none;
  -webkit-mask-image: none;
  animation: none;
}

/* 细碎的白色圆点，呼应哆啦A梦的圆润造型 */
.dora-sky-pattern {
  inset: 0;
  opacity: 0.13;
  background-image:
    radial-gradient(circle, #ffffff 0 2px, transparent 2.5px),
    radial-gradient(circle, #ffffff 0 1.2px, transparent 1.7px);
  background-position:
    0 0,
    23px 23px;
  background-size: 46px 46px;
  mask-image: linear-gradient(180deg, #000 0%, #000 55%, transparent 85%);
  -webkit-mask-image: linear-gradient(180deg, #000 0%, #000 55%, transparent 85%);
}

.dark .dora-sky-pattern {
  opacity: 0.75;
  background-image:
    radial-gradient(circle at 18% 22%, #ffffff 0 1px, transparent 1.6px),
    radial-gradient(circle at 64% 35%, #a5e4ff 0 1px, transparent 1.7px),
    radial-gradient(circle at 34% 78%, #ffd700 0 1.2px, transparent 1.8px),
    radial-gradient(circle at 88% 67%, #ffffff 0 1px, transparent 1.5px);
  background-position: 0 0;
  background-size:
    180px 190px,
    260px 230px,
    310px 290px,
    220px 270px;
  animation: dora-star-twinkle 6s ease-in-out infinite alternate;
}

/* 漫画风白云 */
/* 漫画风白云：从左往右慢慢飘过整片天空，一边飘一边轻轻上下浮动。
   每朵云速度不同；负的 animation-delay 让刷新页面时云已经散落在各自的位置上，
   --cloud-x 是同一个位置在「减少动态效果」时的静止坐标。 */
.dora-cloud {
  left: 0;
  display: block;
  aspect-ratio: 200 / 104;
  background: url('/images/doraemon/cloud.svg') center / contain no-repeat;
  filter: drop-shadow(0 8px 0 rgb(0 90 150 / 0.1));
  animation:
    dora-cloud-float var(--cloud-duration, 110s) linear var(--cloud-delay, 0s) infinite,
    dora-cloud-bob var(--cloud-bob, 6s) ease-in-out infinite alternate;
}

.dora-cloud--1 {
  --cloud-x: 0.03;
  --cloud-duration: 110s;
  --cloud-delay: -14s;
  --cloud-bob: 6.5s;
  top: 11%;
  width: 13rem;
}

.dora-cloud--2 {
  --cloud-x: 0.86;
  --cloud-duration: 95s;
  --cloud-delay: -85s;
  --cloud-bob: 5.2s;
  top: 26%;
  width: 10rem;
}

.dora-cloud--3 {
  --cloud-x: -0.02;
  --cloud-duration: 125s;
  --cloud-delay: -10s;
  --cloud-bob: 7.4s;
  top: 52%;
  width: 8rem;
  opacity: 0.85;
}

.dora-cloud--4 {
  --cloud-x: 0.88;
  --cloud-duration: 118s;
  --cloud-delay: -107s;
  --cloud-bob: 6s;
  top: 62%;
  width: 11rem;
  opacity: 0.8;
}

.dora-cloud--5 {
  --cloud-x: 0.24;
  --cloud-duration: 100s;
  --cloud-delay: -32s;
  --cloud-bob: 5.6s;
  top: 34%;
  width: 12rem;
  opacity: 0.9;
}

.dora-cloud--6 {
  --cloud-x: 0.52;
  --cloud-duration: 112s;
  --cloud-delay: -64s;
  --cloud-bob: 8s;
  top: 68%;
  width: 15rem;
  opacity: 0.85;
}

.dora-cloud--7 {
  --cloud-x: 0.6;
  --cloud-duration: 90s;
  --cloud-delay: -58s;
  --cloud-bob: 4.8s;
  top: 18%;
  width: 9.5rem;
  opacity: 0.9;
}

.dark .dora-cloud {
  opacity: 0.07;
  filter: none;
}

/* 夜里中间这几朵云稍亮一点，透过玻璃能看出轮廓 */
.dark .dora-cloud--5,
.dark .dora-cloud--6,
.dark .dora-cloud--7 {
  opacity: 0.16;
}

/* 哆啦A梦戴着竹蜻蜓，从天空里慢慢飞过 */
.dora-flyer {
  top: 17%;
  left: -7rem;
  width: 4.6rem;
  aspect-ratio: 220 / 272;
  animation: dora-flyer-cross 46s linear infinite;
}

.dora-flyer i {
  display: block;
  width: 100%;
  height: 100%;
  background: url('/images/doraemon/doraemon-fly.svg') center / contain no-repeat;
  filter: drop-shadow(0 10px 8px rgb(0 60 110 / 0.25));
  animation: dora-flyer-bob 2.6s ease-in-out infinite;
}

.dark .dora-flyer {
  opacity: 0.55;
}

/* 小镇天际线 */
.dora-town {
  right: 0;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 220px;
}

.t-far {
  fill: rgb(255 255 255 / 0.2);
}
.t-wall {
  fill: #e4f5ff;
}
.t-roof,
.t-door {
  fill: #0b6fb3;
}
.t-win {
  fill: #86cff5;
}
.t-fence {
  fill: #c4e7fa;
}
.t-fence-line,
.t-rail {
  fill: none;
  stroke: #8ecbeb;
  stroke-width: 2;
}
.t-rail {
  stroke: #0b6fb3;
}
.t-pole {
  fill: #0a5a92;
}
.t-wires path {
  fill: none;
  stroke: rgb(10 90 146 / 0.55);
  stroke-width: 1.4;
}
.t-pipe {
  fill: #d3e6f2;
  stroke: #7fb2d3;
  stroke-width: 2;
}
.t-hole {
  fill: #3b7fab;
}
.t-pipe-rim {
  fill: none;
  stroke: #ffffff;
  stroke-linecap: round;
  stroke-width: 3;
}
.t-grass {
  fill: none;
  stroke: #3fb27f;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 2.5;
}
.t-tree {
  fill: #3fb27f;
}
.t-trunk {
  fill: #7a5530;
}
.t-ground {
  fill: #0a6aa8;
}
.t-road {
  fill: none;
  stroke: rgb(255 255 255 / 0.45);
  stroke-width: 2.5;
}

.dark .t-far {
  fill: rgb(76 195 255 / 0.07);
}
.dark .t-wall {
  fill: #0b2e4f;
}
.dark .t-roof,
.dark .t-door {
  fill: #061a2e;
}
.dark .t-win {
  fill: #15395a;
}
.dark .t-win.t-lit {
  fill: #ffd966;
}
.dark .t-fence {
  fill: #0a2742;
}
.dark .t-fence-line {
  stroke: #0f3656;
}
.dark .t-rail {
  stroke: #061a2e;
}
.dark .t-pole {
  fill: #041424;
}
.dark .t-wires path {
  stroke: rgb(2 12 24 / 0.9);
}
.dark .t-pipe {
  fill: #1c4466;
  stroke: #0f2f4c;
}
.dark .t-hole {
  fill: #04121f;
}
.dark .t-pipe-rim {
  stroke: rgb(165 228 255 / 0.25);
}
.dark .t-grass,
.dark .t-tree {
  fill: #0d3b3b;
  stroke: #0d3b3b;
}
.dark .t-grass {
  fill: none;
}
.dark .t-trunk {
  fill: #06182b;
}
.dark .t-ground {
  fill: #031120;
}
.dark .t-road {
  stroke: rgb(255 215 0 / 0.2);
}

@media (max-width: 640px) {
  .dora-town {
    width: calc(100% / 0.62);
    transform: scale(0.62);
    transform-origin: left bottom;
  }

  .dora-kids {
    transform: scale(0.62);
    transform-origin: left bottom;
  }

  .dora-sun {
    top: -18rem;
    left: -16rem;
  }

  .dark .dora-sun {
    top: -6rem;
    left: -6rem;
  }

  .dora-cloud--1 {
    width: 8rem;
  }

  .dora-cloud--2,
  .dora-cloud--4 {
    width: 6.5rem;
  }

  .dora-cloud--5,
  .dora-cloud--6 {
    width: 7.5rem;
  }

  .dora-cloud--7 {
    width: 6rem;
  }

  .dora-flyer {
    width: 3.2rem;
  }
}

/* 空地上抛球的大雄和静香：3.2 秒一个来回，球飞到谁那边，谁就举手去接、轻轻一跳 */
.dora-kids {
  bottom: 0;
  left: 0;
  overflow: visible;
}

.k-skin {
  fill: #ffdcbc;
}
.k-hair {
  fill: #1d1d24;
}
.k-shirt {
  fill: #ffd23f;
}
.k-shorts {
  fill: #26418f;
}
.k-shoe-n {
  fill: #6b4226;
}
.k-dress {
  fill: #ff8fb3;
}
.k-shoe-s {
  fill: #e0405a;
}
.k-tie {
  fill: #e0405a;
}
.k-eye {
  fill: #1d1d24;
}
.k-blush {
  fill: #ff9aa8;
  opacity: 0.7;
}
.k-mouth,
.k-glasses {
  fill: none;
  stroke: #1d1d24;
  stroke-linecap: round;
  stroke-width: 1;
}
.k-glasses circle {
  fill: rgb(255 255 255 / 0.35);
}
.k-ball {
  fill: #ff4d5a;
  stroke: #ffffff;
  stroke-width: 1;
}
.k-ball-stripe {
  fill: none;
  stroke: #ffffff;
  stroke-width: 1.4;
}

.kid,
.kid__arm,
.kid__tail {
  transform-box: fill-box;
}

.kid {
  transform-origin: 50% 100%;
}

.kid__arm {
  transform-origin: 50% 8%;
}

.kid__tail {
  transform-origin: 50% 0;
}

.kid--nobita {
  animation: kid-hop-nobita 3.2s ease-in-out infinite;
}
.kid--nobita .kid__arm--front {
  animation: kid-arm-nobita 3.2s ease-in-out infinite;
}
.kid--nobita .kid__arm--back {
  animation: kid-arm-back 1.6s ease-in-out infinite alternate;
}

.kid--shizuka {
  animation: kid-hop-shizuka 3.2s ease-in-out infinite;
}
.kid--shizuka .kid__arm--front {
  animation: kid-arm-shizuka 3.2s ease-in-out infinite;
}
.kid--shizuka .kid__arm--back {
  animation: kid-arm-back 1.6s ease-in-out infinite alternate-reverse;
}
.kid__tail--left {
  animation: kid-tail 1.6s ease-in-out infinite alternate;
}
.kid__tail--right {
  animation: kid-tail 1.6s ease-in-out infinite alternate-reverse;
}

.kid-ball {
  animation: kid-ball-x 1.6s linear infinite alternate;
}
.kid-ball__arc {
  animation: kid-ball-y 1.6s infinite;
}

.dark .dora-kids {
  filter: brightness(0.62) saturate(0.85);
}

@keyframes kid-ball-x {
  to {
    transform: translateX(48px);
  }
}

@keyframes kid-ball-y {
  0% {
    transform: translateY(0);
    animation-timing-function: cubic-bezier(0.2, 0.6, 0.4, 1);
  }
  50% {
    transform: translateY(-42px);
    animation-timing-function: cubic-bezier(0.6, 0, 0.8, 0.4);
  }
  100% {
    transform: translateY(0);
  }
}

/* 大雄的前手：0% 刚把球抛出去，88%～100% 举起来接回来的球 */
@keyframes kid-arm-nobita {
  0%,
  100% {
    transform: rotate(-130deg);
  }
  14% {
    transform: rotate(-60deg);
  }
  28%,
  74% {
    transform: rotate(-12deg);
  }
  88% {
    transform: rotate(-100deg);
  }
}

/* 静香的前手：50% 接到球，紧接着抛回去 */
@keyframes kid-arm-shizuka {
  0%,
  24%,
  78%,
  100% {
    transform: rotate(12deg);
  }
  38% {
    transform: rotate(100deg);
  }
  50% {
    transform: rotate(130deg);
  }
  64% {
    transform: rotate(60deg);
  }
}

@keyframes kid-arm-back {
  from {
    transform: rotate(-14deg);
  }
  to {
    transform: rotate(14deg);
  }
}

@keyframes kid-hop-nobita {
  0%,
  86%,
  100% {
    transform: translateY(0);
  }
  93% {
    transform: translateY(-5px);
  }
}

@keyframes kid-hop-shizuka {
  0%,
  36%,
  54%,
  100% {
    transform: translateY(0);
  }
  43% {
    transform: translateY(-5px);
  }
}

@keyframes kid-tail {
  from {
    transform: rotate(-10deg);
  }
  to {
    transform: rotate(10deg);
  }
}

@keyframes dora-sun-spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes dora-star-twinkle {
  to {
    opacity: 0.45;
  }
}

@keyframes lg-glow-drift {
  to {
    transform: translate3d(-6vw, 4vh, 0) scale(1.12);
  }
}

/* 游走用独立的 translate、呼吸用 scale + opacity，两组动画可以叠加而不互相覆盖 */
@keyframes lg-glow-wander-gold {
  0%,
  100% {
    translate: 0 0;
  }
  25% {
    translate: -9vw 7vh;
  }
  50% {
    translate: -16vw -2vh;
  }
  75% {
    translate: -5vw -6vh;
  }
}

@keyframes lg-glow-wander-pink {
  0%,
  100% {
    translate: 0 0;
  }
  25% {
    translate: 11vw -6vh;
  }
  50% {
    translate: 5vw -13vh;
  }
  75% {
    translate: 14vw 4vh;
  }
}

@keyframes lg-glow-breathe {
  from {
    opacity: 0.7;
    scale: 0.9;
  }
  to {
    opacity: 1;
    scale: 1.16;
  }
}

@keyframes lg-glow-orbit {
  to {
    rotate: 360deg;
  }
}

@keyframes dora-cloud-float {
  from {
    transform: translate3d(-100%, 0, 0);
  }
  to {
    transform: translate3d(100vw, 0, 0);
  }
}

@keyframes dora-cloud-bob {
  from {
    translate: 0 0;
  }
  to {
    translate: 0 -10px;
  }
}

@keyframes dora-flyer-cross {
  0% {
    transform: translate3d(0, 0, 0) rotate(8deg);
  }
  25% {
    transform: translate3d(28vw, -30px, 0) rotate(4deg);
  }
  50% {
    transform: translate3d(56vw, 16px, 0) rotate(10deg);
  }
  75% {
    transform: translate3d(84vw, -20px, 0) rotate(5deg);
  }
  100% {
    transform: translate3d(calc(100vw + 14rem), 0, 0) rotate(8deg);
  }
}

@keyframes dora-flyer-bob {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-6px);
  }
}

@media (prefers-reduced-motion: reduce) {
  .lg-glow,
  .lg-glow::before,
  .dora-sun,
  .dora-cloud,
  .dora-flyer,
  .dora-flyer i,
  .dark .dora-sky-pattern {
    animation: none;
  }
  .dora-flyer {
    display: none;
  }
  .dora-cloud {
    transform: translate3d(calc(var(--cloud-x, 0) * 100vw), 0, 0);
  }
  .kid,
  .kid__arm,
  .kid__tail,
  .kid-ball,
  .kid-ball__arc {
    animation: none;
  }
}

.background-loading {
  position: absolute;
  inset: 0;
  background-color: #04213b;
}

:root:not(.dark) .background-loading {
  background: linear-gradient(180deg, #33b5f0, #0081c9);
}

.background-media {
  position: absolute;
  inset: 0;
  transition: opacity 0.8s ease;
}

.background-image {
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.background-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.background-overlay {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
