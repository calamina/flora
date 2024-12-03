<script setup lang="ts">
import { onMounted, ref, type Ref } from 'vue'
import { useDateFormat } from '@vueuse/core'
import { onKeyStroke } from '@vueuse/core'
import media from '@/data/media.json'
import { gsap } from "gsap";

enum view { home, media, news }
const active = ref(view.home)
const news: Ref<any> = ref([])
const meteo: Ref<any> = ref(null)
const mediaList: Ref<any> = ref(media)

onKeyStroke('ArrowLeft', () => {
  active.value = active.value === view.media ? view.home : view.media
})
onKeyStroke('ArrowRight', () => {
  active.value = active.value === view.news ? view.home : view.news
})

onMounted(() => {
  getNews()
  getMeteo()
})

function onEnter(el: Element, done: () => void) {
  if (active.value === view.home) {
    gsap.set(el, {
      left: '-2rem'
    })
    gsap.to(el, {
      left: 0,
      duration: 0.3,
      onComplete: done
    })
  }
  else if (active.value === view.media) {
    gsap.set(el, {
      bottom: '-100vh'
    })
    gsap.to(el, {
      bottom: 0,
      duration: 0.3,
      onComplete: done
    })
  }
  else {
    gsap.set(el, {
      top: '-100vh'
    })
    gsap.to(el, {
      top: 0,
      duration: 0.3,
      onComplete: done
    })
  }
}

function onLeave(el: Element, done: () => void) {
  const element = el.classList[0]
  if (element === 'linksWrap') {
    gsap.to(el, {
      left: '-2rem',
      opacity: 0,
      duration: 0.3,
      onComplete: done
    })
  } else if (element === 'mediaWrap') {
    gsap.to(el, {
      bottom: '-100vh',
      duration: 0.3,
      onComplete: done
    })
  } else {
    gsap.to(el, {
      top: '-100vh',
      duration: 0.3,
      onComplete: done
    })
  }
}

const getNews = async () => {
  const response = await fetch(`https://api.nytimes.com/svc/topstories/v2/world.json?api-key=${import.meta.env.VITE_NEWS}`);
  const res = await response.json();
  for (var i = 0; i < 10; i++) {
    var obj = res.results[i];
    const date = useDateFormat(new Date(obj.published_date), 'DD/MM ~ HH:MM').value
    news.value.push({ date, title: obj.title, url: obj.url })
  }
}

const getMeteo = async () => {
  const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?id=2996944&appid=${import.meta.env.VITE_METEO}&units=metric`)
  // alt in case
  // const response = await fetch('https://api.open-meteo.com/v1/forecast?latitude=45.7679&longitude=4.8506&current=temperature_2m,weather_code&timezone=GMT&models=meteofrance_seamless')
  const res = await response.json()
  meteo.value = {
    temp: Math.round(res.main.temp),
    desc: res.weather["0"].description,
  }
}
</script>

<template>
  <main>
    <transition @enter="onEnter" @leave="onLeave">
      <nav v-if="active === view.home" class="linksWrap">
        <a href="https://www.gmail.com"><span></span>mail</a>
        <a href="https://www.youtube.com"><span></span>youtube</a>
        <a href="https://www.senscritique.com"><span></span>senscritique</a>
        <a href="https://www.facebook.com"><span></span>facebook</a>
        <a href="https://www.univ-lyon2.fr"><span></span>uni</a>
      </nav>

      <div v-else-if="active === view.news" class="newsWrap">
        <p v-for="item in news" :key="item.title">
          [{{ item.date }}]
          <a :href="item.url" target="_blank"><span class="script">{{ item.title }}</span> ⤴</a>
        </p>
      </div>

      <div v-else-if="active === view.media" class="mediaWrap">
        <ul v-for="media in mediaList">
          <li class="title">{{ media.title }}</li>
          <li v-for="element in media.content">{{ element }}</li>
        </ul>
      </div>
    </transition>

    <div class="toggle">
      <button @click="active = active === view.media ? view.home : view.media">media</button>
      <span>/</span>
      <button @click="active = active === view.news ? view.home : view.news">news</button>
    </div>

    <div class="meteo" v-if="meteo">
      <p>{{ meteo.temp }}<span>°C</span></p>
      <p>{{ meteo.desc }}</p>
    </div>

  </main>
</template>

<style scoped lang="scss">
main {
  height: 100vh;
}

.toggle {
  z-index: 999;
  position: fixed;
  bottom: 2rem;
  right: 3rem;

  button {
    font-family: inherit;
    background: none;
    color: #fff;
    outline: none;
    border: none;
    margin-block-start: 0;
    margin-block-end: 0;
    line-height: 4.25rem;
    font-size: 3rem;
    text-align: right;
    cursor: pointer;
    text-shadow: 0px 0px 15px #121212;
    text-decoration: underline;
    text-decoration-color: transparent;
    transition: padding .3s ease, text-decoration-color .3s ease;

    &:active,
    &:hover {
      padding: 0 2rem;
      text-decoration-color: rgba($color: #fff, $alpha: 0.1);
      text-decoration: underline;
    }
  }

  span {
    color: #fff;
    opacity: .1;
    line-height: 4.25rem;
    font-size: 3rem;
    text-align: center;
    pointer-events: none;
  }
}

.meteo {
  z-index: 80;
  position: absolute;
  top: 0;
  right: 0;
  padding: 4rem;
  transition: opacity .3s ease, transform .3s ease;

  p {
    margin-block-start: 0;
    margin-block-end: 0;
    line-height: 4.25rem;
    font-size: 5rem;
    text-align: right;
  }

  span {
    color: #ffffff15;
  }
}

.linksWrap {
  position: fixed;
  left: 0;
  z-index: 50;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  padding-left: 2.5rem;
  overflow: hidden;

  a {
    display: flex;
    align-items: center;
    color: #ffffff;
    font-size: 9vw;
    line-height: 8vw;
    height: 8vw;
    width: fit-content;
    transition: padding .3s ease, opacity .3s ease, text-decoration-color .3s ease;
    text-shadow: 0px 0px 20px #121212;
    text-decoration-color: #ffffff00;
    position: relative;
    pointer-events: none;

    span {
      top: 2vh;
      display: block;
      position: absolute;
      width: 100%;
      height: 100%;
      pointer-events: auto;
    }

    &::before,
    &::after {
      color: #ffffff00;
      transition: opacity .3s;
      text-shadow: none;
    }

    &::before {
      content: ".";
    }

    &::after {
      content: "*";
    }

    &:hover {
      padding-left: 2rem;
      text-decoration: underline;
      text-decoration-color: #ffffff15;

      &::before,
      &::after {
        color: #ffffff15;
      }
    }
  }

  .afterlink {
    opacity: .1;

    &::before {
      content: "";
      margin-left: -2rem;
      line-height: 5rem;
    }
  }
}

.mediaWrap,
.newsWrap {
  z-index: 100;
  position: absolute;
  height: 100vh;
  width: 100vw;
  background: url(./assets/bg-min-blur.jpg) no-repeat;
  background-size: cover;
  background-attachment: fixed;
}

.mediaWrap {
  box-sizing: border-box;
  display: flex;
  flex-flow: column wrap;
  align-items: flex-start;
  gap: 1rem 3rem;
  padding: 3rem;

  ul {
    list-style-type: none;
    padding: 0;
    overflow: hidden;
    // max-width: 25rem;

    li {
      width: 100%;
      font-family: "Apercu mono";
      font-size: 1.5rem;
      line-height: 2rem;
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;

      &.title {
        font-style: italic;
        color: #a580be !important;
        padding-bottom: 1rem;
      }
    }
  }
}


.newsWrap {
  box-sizing: border-box;
  flex-direction: column;
  padding: 3rem;
  display: flex;
  flex-direction: column;
  // align-items: flex-start;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;

  p {
    display: flex;
    align-items: center;
    margin-block-start: 0;
    margin-block-end: 0;
    font-family: "Apercu mono";
    color: #a580be;
    font-style: italic;
    font-size: 1.5rem;
    font-weight: 400;
    line-height: 3rem;
    overflow: hidden;
    white-space: nowrap;
    max-width: 90rem;
    width: 100%;
    gap: 1.5rem;
    height: 2rem;
    
    &:hover span {
      text-decoration: underline;
      }
      }
      
      a {
        display: flex;
        align-items: center;
        color: #fff;
        text-decoration: none;
        overflow: hidden;
        white-space: nowrap;
        gap: 1rem;

    span {
      display: block;
      color: #fff;
      font-style: normal;
      letter-spacing: -0.05rem;
      font-size: 1.5rem;
      font-weight: 500;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
}
</style>
