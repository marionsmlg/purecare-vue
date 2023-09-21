<script setup>
import Category from './Category.vue'
import { ref, computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const recipeCategoryName = route.params.category
console.log(recipeCategoryName)

const hairTypeId = ref(localStorage.getItem('hairType') || '')
const skinTypeId = ref(localStorage.getItem('skinType') || '')
const strOfHairProblemId = ref(localStorage.getItem('hairProblem') || '')
const strOfSkinProblemId = ref(localStorage.getItem('skinProblem') || '')
const arrOfSkinProblemId = JSON.parse(strOfSkinProblemId.value)
const arrOfHairProblemId = JSON.parse(strOfHairProblemId.value)

const skinRecipes = ref([])
const hairRecipes = ref([])

let page = 1
let limit = 9

function displayNextRecipes() {
  page++
  if (recipeCategoryName === 'cheveux') {
    if (hairRecipes.value.length === limit) {
      limit = 9 * page

      fetchHairRecipeByHairTypeId()
    }
  } else {
    if (skinRecipes.value.length === limit) {
      limit = 9 * page
    }
    fetchSkinRecipeBySkinTypeId()
  }
}

const canDisplayMoreRecipes = computed(() => {
  if (recipeCategoryName === 'cheveux') {
    console.log(hairRecipes.value.length)
    console.log({ limit })
    console.log(hairRecipes.value.length >= limit)
    return hairRecipes.value.length > limit
  } else {
    return skinRecipes.value.length > limit
  }
})

async function fetchSkinRecipeBySkinTypeId() {
  const queryParamsSkinType = new URLSearchParams({
    physical_trait_id: `${skinTypeId.value},b9f90678-ea3f-4fde-952f-a26a88e13259`,
    beauty_issue_id: arrOfSkinProblemId.join(','),
    limit: limit
  })
  try {
    const apiUrl = `http://localhost:3000/api/recipe?${queryParamsSkinType}`
    const response = await fetch(apiUrl)
    const recipes = await response.json()
    skinRecipes.value = recipes
    console.log(recipes)
  } catch (error) {
    console.error(error)
  }
}

async function fetchHairRecipeByHairTypeId() {
  const queryParamsHairType = new URLSearchParams({
    physical_trait_id: `${hairTypeId.value},c8898a24-04cb-4b1f-bb8b-38633aa3c670`,
    beauty_issue_id: arrOfHairProblemId.join(','),
    limit: limit
  })
  try {
    const apiUrl = `http://localhost:3000/api/recipe?${queryParamsHairType}`
    const response = await fetch(apiUrl)
    const recipes = await response.json()
    hairRecipes.value = recipes
    console.log(recipes)
  } catch (error) {
    console.error(error)
  }
}

fetchSkinRecipeBySkinTypeId()
fetchHairRecipeByHairTypeId()
</script>

<template>
  <Category
    :recipes="recipeCategoryName === 'cheveux' ? hairRecipes : skinRecipes"
    :categoryName="recipeCategoryName"
  />
  <div class="flex justify-center" v-if="canDisplayMoreRecipes">
    <button
      @click="displayNextRecipes"
      type="button"
      class="md:px-14 md:py-3 w-auto rounded-xl bg-[#8CD4E0] px-3 py-2 text-md font-bold shadow-sm hover:bg-[#1AB2CC] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
    >
      Afficher plus de recettes
    </button>
  </div>
</template>