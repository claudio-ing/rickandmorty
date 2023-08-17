<template>
  <div class="full-container full-height">
    <div class="d-flex h-15">
      <h2>Results:{{ info?.count }}, Pages: {{ info?.pages }}</h2>
    </div>
    <div class="d-flex h-85">
      <div :class="layoutWidth" class="flex-container scroll-y" v-if="results">
        <div v-for="character in results" :key="character.id">
          <ul>
            <li>
              <a @click="characterSelected(character)">{{ character.name }}</a>
            </li>
            <li>{{ character.species }}, {{ character.gender }}</li>
            <li><img width="60" height="60" :src="character.image" /></li>
            <li>
              Episodes:
              <span v-for="(episode, index) in character.episode.slice(0, 5)" :key="episode" >
                <a @click="episodeSelected(episode)">{{ getIdFromUrl(episode) }}</a>
                <span v-if="(index != 4) && (index + 1) < character.episode.length">, </span>
              </span>
              <span v-if="character.episode.length > 5">, ...</span>
            </li>
          </ul>
        </div>
      </div>
      <div class="w-30 scroll-y" v-if="layoutWidth != 'full-width'">
        <div class="character" v-if="selectedCharacter">
          <CharacterDetail :character="selectedCharacter"/>
        </div>
        <div class="episode" v-if="selectedEpisode">
          <EpisodeDetail :episode="selectedEpisode" @update:character="onUpdateCharacter"  />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, Ref } from 'vue'
import { Character, Episode } from '@/interfaces'
import { CharacterStore, useCharacterStore } from '@/store/characters'
import { EpisodeStore, useEpisodeStore } from '@/store/episodes'
import getIdFromUrl from '@/common/utils'
import CharacterDetail from '@/components/CharacterDetail.vue'
import EpisodeDetail from '@/components/EpisodeDetail.vue'

const selectedCharacter: Ref<Character|null> = ref(null)
const selectedEpisode: Ref<Episode|null> = ref(null)
const page: Ref<number> = ref(1)

let episodeStore: EpisodeStore
// TODO SET FILTER
const chStore: CharacterStore = await useCharacterStore(page.value)
const info = chStore.response?.info
const results = chStore.response?.results

const layoutWidth = computed((): string =>
  (selectedCharacter.value === null && selectedEpisode.value == null) ? 'full-width' : 'w-70')

function characterSelected (ch: Character) {
  selectedCharacter.value = (ch.id === selectedCharacter.value?.id) ? null : ch
}

async function onUpdateCharacter (id: number) {
  const character = await chStore.fetchCharacterData(id)
  if (character) {
    characterSelected(character)
  }
}

async function episodeSelected (ep: string) {
  const episode: number = getIdFromUrl(ep)
  if (episode === selectedEpisode.value?.id) {
    selectedEpisode.value = null
    return
  }
  episodeStore = await useEpisodeStore(episode)
  selectedEpisode.value = episodeStore.episodeData[episode]!
}

</script>

<style scoped>

.full-container {
  max-height: 97%;
  display: flex;
  flex-direction: column;
  margin-bottom: 3%
}

.h-15 {
  height: 15%;
}

.h-85 {
  height: 85%;
}

.flex-container {
  display: flex;
  flex-wrap: wrap;
  margin-left: -5px;
}
.flex-container > div {
  width: 160px;
  height: 160px;
  margin: 5px;
  border: 1px solid #ccc;
}

.full-height {
  height: 100%;
}

.full-width {
  width: 100%;
}

.w-70 {
  width: 70%;
}

.w-30 {
  width: 30%;
}

</style>