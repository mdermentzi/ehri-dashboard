<template>
  <div class="grid grid-cols-12 lg:grid-cols-8 gap-2 lg:gap-4 h-screen max-w-full">
    <!-- SMALL SCREEN UI -->
    <div v-if="!isLargeScreen && !expandFocusedItem" class="lg:hidden h-screen col-span-11 shadow-xl bg-white overflow-hidden px-7">
      <h4 class="font-sans text-ehri-dark font-extralight text-xl mt-4">Showing <span class="font-serif font-extrabold">{{total}}</span> {{ total>1?'Archival Institutions':'Archival Institution' }}</h4>
      <p class="font-sans text-ehri-dark text-xs text-justify pb-4">{{ desc }}</p>
      <div class="h-full flex flex-col" >
        <ul ref="el" class="h-5/6 overflow-scroll">
          <ArchivalInstitutionItemCard v-for="item of items" :key="item.id" :repoObject="item" :selectedItem="selectedRepoID" @idChange="(id)=>changeRepoID(id)"></ArchivalInstitutionItemCard>
          <li v-if="loading" class="w-full flex justify-center items-center py-2">
            <LoadingComponent></LoadingComponent>
          </li>
        </ul>
      </div>
    </div>
    <div :class="[focusedItemClass,'flex', 'h-screen', 'lg:hidden', 'shadow-xl', ]">
      <span v-if="!expandFocusedItem" class="cursor-pointer flex justify-center align-middle items-center" @click="toggleFocusedItemClass">
        <span
            class="material-symbols-outlined text-white pointer-events-none"
        >
          chevron_left
        </span>
      </span>
      <div v-else class="overflow-hidden">
        <span class="cursor-pointer bg-ehri-dark text-white px-5 py-1" @click="toggleFocusedItemClass">
          <span
            class="pt-3 material-symbols-outlined pointer-events-none align-bottom"
          >
            close
          </span>
          Close
        </span>
        <h4 class="font-sans text-ehri-dark font-extralight text-xl mt-4">Item Details:</h4>
        <ArchivalInstitutionDetails v-if="selectedRepoID" :selectedRepoID="selectedRepoID"></ArchivalInstitutionDetails>
      </div>
    </div>


    <!-- LARGE SCREEN UI -->
    <div v-if="isLargeScreen" class="hidden lg:block shadow-xl bg-white lg:h-4/5 lg:col-span-3 overflow-hidden px-7">
      <h4 class="font-sans text-ehri-dark font-extralight text-xl mt-4">Showing <span class="font-serif font-extrabold">{{total}}</span> {{ total>1?'Archival Institutions':'Archival Institution' }}</h4>
      <p class="font-sans text-ehri-dark text-xs text-justify pb-4">{{ desc }}</p>
      <div class="h-4/6 flex flex-col" >
        <ul ref="el" class="hidden lg:block overflow-y-scroll">
          <ArchivalInstitutionItemCard v-for="item of items" :key="item.id" :repoObject="item" :selectedItem="selectedRepoID" @idChange="(id)=>changeRepoID(id)"></ArchivalInstitutionItemCard>
          <li v-if="loading" class="w-full flex justify-center items-center py-2">
            <LoadingComponent></LoadingComponent>
          </li>
        </ul>
      </div>
    </div>
    <div class="hidden lg:block lg:col-span-5 bg-white shadow-xl h-4/5 pb-7 px-7">
      <h4 class="font-sans text-ehri-dark font-extralight text-xl mt-4">Item Details:</h4>
      <ArchivalInstitutionDetails v-if="selectedRepoID" :selectedRepoID="selectedRepoID"></ArchivalInstitutionDetails>
    </div>
  </div>
</template>

<script>
import { ref, toRef, onMounted, computed } from "vue";
import ArchivalInstitutionDetails from "./ArchivalInstitutionDetails.vue";
import ArchivalInstitutionItemCard from "./ArchivalInstitutionItemCard.vue";
import { useInfiniteScroll } from '@vueuse/core'
import {fetchFacetedPortalSearch} from "../services/EHRIGetters";
import LoadingComponent from "./LoadingComponent.vue";

export default {
  name: "ArchivalInstitutionItems",
  components: {
    ArchivalInstitutionItemCard,
    ArchivalInstitutionDetails, LoadingComponent },
  props: {
    searchTerm: String,
    holder: String,
    type: String,
    desc: String
  },
  setup(props) {
    const items = ref([])
    const repoQuery = toRef(props, 'searchTerm')
    const holderFilter = toRef(props, 'holder')
    const typeFilter = toRef(props, 'type')
    const desc = toRef(props, 'desc')
    const page = ref(1)
    const el = ref(null)
    const filters = ref(new Object())
    const total = ref()
    const loading = ref(false)
    const expandFocusedItem = ref(false);
    const screenWidth = ref(window.innerWidth);

    const isLargeScreen = computed(() => {
      return screenWidth.value >= 1024;
    });

    window.addEventListener('resize', () => {
      screenWidth.value = window.innerWidth;
    });


    holderFilter.value ? filters.value['holder']=encodeURIComponent(holderFilter.value) : null
    typeFilter.value ? filters.value['type']= typeFilter.value: null

    const selectedRepoID = ref("")

    const focusedItemClass = computed(() => {
            return expandFocusedItem.value
            ? "w-full h-screen bg-white text-ehri-purple px-7 overflow-y-scroll col-span-12 m-0 transition-all ease-in-out duration-800 pb-3"
            : "w-full bg-ehri-dark col-span-1 transition-all ease-in-out  duration-800";
        });

   
    const toggleFocusedItemClass = () => {
      expandFocusedItem.value = !expandFocusedItem.value;
    };

    const changeRepoID = (id)=> {
      selectedRepoID.value = id
      expandFocusedItem.value = true
    }


    const getUnitsOnScroll = async () => {
      loading.value = true;
      const newUnits = await fetchFacetedPortalSearch(repoQuery.value, page.value, filters.value, 25)
      newUnits.data.data.forEach(newItem => {
        if (!items.value.some(item => item.id === newItem.id)) {
          items.value.push(newItem)
        }
      })
      total.value = newUnits.data.meta.total
      page.value++
      loading.value = false;
    }

    useInfiniteScroll(
      el,
      async () => {
        await getUnitsOnScroll()
      },
      { distance: 300 }
    )


    onMounted(()=>{
      getUnitsOnScroll()
      .then(()=>{
        selectedRepoID.value = items.value.length>0?items.value[0].id:null
      })
    })
    return { isLargeScreen, expandFocusedItem, focusedItemClass, toggleFocusedItemClass, total, items, el,  loading, desc, selectedRepoID, changeRepoID };
  },
};
</script>
