<template>
    <span class="flex justify-center bg-ehri-purple py-2 text-white lg:hidden col-span-12 text-ehri-dark font-sans " @click="toggleFilterBar">
      <span v-if="!showFilterBar" class="lg:hidden mr-2 cursor-pointer">
              <span
            class="material-symbols-outlined text-ehri-white pointer-events-none align-bottom"
          >
            filter_alt
          </span>
          Filter Results
      </span>
      <span v-else class="lg:hidden mr-2 cursor-pointer">
              <span
            class="material-symbols-outlined text-ehri-white pointer-events-none align-bottom"
          >
            close
          </span>
          Close
      </span>
  </span>
  <div v-if="!loading && !showFilterBar && selectedEdition.isFiltered()" class="lg:hidden pt-1 px-2">
            <h5 class="font-sans text-sm text-ehri-purple block font-medium">Active Filters:</h5>
              <div class="" v-for="f in Object.entries(selectedEdition['filters'])">
                <span v-if="f[1]!==''" class="inline mt-1 w-fit cursor-pointer border border-ehri-dark rounded-full bg-ehri-dark text-white mr-1 px-2 py-0.5 text-xs"
                  @click="handleFilter('',f[0],selectedEdition['edition'])">{{f[1]}}</span>
              </div>
              <div class="flex items-center mt-1 cursor-pointer text-ehri-purple text-sm" id="remove-filter" @click="() => handleFilter('','removeAll',selectedEdition['edition'])">
                <span class="material-symbols-outlined pointer-events-none w-3 h-3 text-xs mr-1">
                  close
                </span>
                <span>Remove All Filters</span>
              </div>
  </div>
  <div class="grid grid-cols-12 sm:grid-cols-8 gap-4 h-screen max-w-full"  v-if="!loading">
    <div class="h-screen col-span-12 bg-white shadow-xl lg:h-4/5 lg:col-span-6 overflow-hidden px-7">
      <h4 class="font-sans text-ehri-dark font-extralight text-xl mt-4">Showing 
        <span v-if="selectedEdition.pagination.total" class="font-serif font-extrabold">{{selectedEdition.pagination.total}}</span>
        <LoadingComponent v-else></LoadingComponent> <span>{{ selectedEdition.pagination.total>1?' results':' result' }}</span>
        from the {{ selectedEdition['title'] }}</h4>
      <p class="font-sans text-ehri-dark text-sm">{{ selectedEdition['description'] }}</p>
      <hr class="text-ehri-dark border-4 shadow-md mt-3">
      <div class="pl-3 pr-1 pt-3 pb-0 h-full">
        <div class="h-5/6">
          <div class="h-full flex flex-col">
            <ul  v-if="selectedEdition && !selectedEdition.isFiltered()" ref="el" class="h-5/6 overflow-scroll">
              <DigitalEditionItem v-for="item of selectedEdition['items']" :key="item.id" :editionObject="item" :edition-string="selectedEdition['edition']"></DigitalEditionItem>
              <li v-if="selectedEdition.loading" class="w-full flex justify-center items-center py-2">
                <LoadingComponent></LoadingComponent>
              </li>
            </ul>
            <ul v-else ref="el" class="h-full overflow-scroll">
              <DigitalEditionItem v-for="item of selectedEdition['filteredItems']" :key="item.id" :editionObject="item" :edition-string="selectedEdition['edition']"></DigitalEditionItem>
              <li v-if="selectedEdition.loading" class="w-full flex justify-center items-center py-2">
                <LoadingComponent></LoadingComponent>
              </li>
            </ul>
        </div>
      </div>
    </div>
    </div>
      <div :class="[filterBarClass, 'bg-ehri-purple', 'overflow-scroll', 'text-white','lg:text-ehri-dark', 'lg:col-span-2', 'col-span-12', 'lg:order-last', 'order-first', 'lg:bg-white', 'shadow-xl', 'lg:h-4/5',]">
        <div class="h-full px-4 pt-4">
          <h4 class="uppercase font-serif font-bold text lg:text-ehri-dark">Filters</h4>
          <p class="font-sans text-sm font-light mb-4">Choose one or more filters</p>
          <div class="overflow-scroll h-full">
            <h5 class="mt-3 uppercase font-serif font-bold text-sm lg:text-ehri-dark">ONLINE EDITION</h5>
          <select v-if="sortedDEResultsNonNull" @change="(e)=>selectEdition(e)" class="text-ehri-dark font-sans text-xs p-1 font-light border border-[1.5px] border-ehri-dark w-full" size="1" aria-label="Online Edition Filter">
            <option v-for="i in sortedDEResultsNonNull" :key="i[0]" :value="i[0]" :selected="selectedEdition['edition'] === i[0]">{{ i[1]['title']+" (" +i[1]['pagination']['total']+")" }} </option>
          </select>
          <div
              v-for="f in Object.entries(selectedEdition['facets'])">
            <DigitalEditionsFilter
                v-if="Object.keys(f[1]).length!==0&&f[0]!=='Featured'"
                :key="filterKey"
                :filter-name="f[0]"
                :filter-object="f[1]"
                @filterChange="(e) => {
                  handleFilter(e, f[0], selectedEdition['edition'])
                }"
            >
            </DigitalEditionsFilter>
          </div>
          <div v-if="selectedEdition.isFiltered()" class="px-4 pt-4">
            <hr class="py-1 lg:text-ehri-dark">
            <h5 class="font-serif text-sm font-extralight">Active Filters:</h5>
              <div class="" v-for="f in Object.entries(selectedEdition['filters'])">
                <span v-if="f[1]!==''" class="block mt-1 w-fit cursor-pointer border rounded-full lg:border-ehri-dark bg-white text-ehri-dark lg:bg-ehri-dark lg:text-white mr-1 px-2 py-0.5 text-xs"
                  @click="handleFilter('',f[0],selectedEdition['edition'])">{{f[1]}}</span>
              </div>
              <div class="flex items-center my-2 cursor-pointer lg:text-ehri-wine text-sm" id="remove-filter" @click="() => handleFilter('','removeAll',selectedEdition['edition'])">
                <span class="material-symbols-outlined pointer-events-none w-3 h-3 text-xs mr-1">
                  close
                </span>
                <span>Remove All Filters</span>
              </div>
          </div>
          <div class="h-2/3">
            <Chart
          class="w-full pb-2"
              :key="chartKey"
              v-if="selectedEdition.isFiltered()?selectedEdition['filteredItems'].length:selectedEdition['items'].length"
              :dataset="selectedEdition['facets']['Subject']">
          </Chart>
          </div>
          </div>
        </div>
      </div>
    </div>
    <LoadingComponent v-else></LoadingComponent>
</template>

<script>
import { toRef, ref, watch, computed } from "vue";
import {
  fetchETEitems,
  fetchDRitems,
  fetchBGFitems,
} from "../services/EHRIGetters.js";
import LoadingComponent from "./LoadingComponent.vue";
import DigitalEditionsFilter from "./DigitalEditionsFilter.vue";
import Chart from "./DigitalEditionsChart.vue";
import { useInfiniteScroll } from '@vueuse/core'
import DigitalEditionItem from "./DigitalEditionItem.vue"

export default {
  name: "DigitalEditions",
  props: {
    searchTerm: String,
  },
  components: {Chart, DigitalEditionItem, LoadingComponent, DigitalEditionsFilter},
  setup(props) {
    const searchTerm = toRef(props, "searchTerm");
    const sortedDEResults = ref([])
    const sortedDEResultsNonNull = ref([])
    const selectedEdition = ref({})
    const chartKey = ref(0)
    const el = ref(null)
    const filterKey = ref(0)
    const showFilterBar = ref(false);

    const DigitalEditionsData = ref({
      ETE: {
        edition: "ETE",
        title: "Early Holocaust Testimony Edition",
        description: "This edition, for the first time, brings together samples of early testmonies of Jewish witnesses and survivors taken before the 1960s.",
        pagination:{
          total: null,
        },
        page: 1,
        items: [],
        filteredItems: [],
        facets: {},
        filters: {
          Archive: "",
          Creator: "",
          Subject: "",
          Person: "",
          Organisation: "",
          Place: ""
        },
        loading: false,
        getUnitsOnScroll: () => {
          DigitalEditionsData.value.ETE.loading = true;
          fetchETEitems(searchTerm.value, DigitalEditionsData.value.ETE.page, 10, DigitalEditionsData.value.ETE.filters)
          .then((newUnits)=>{
            if(Object.values(DigitalEditionsData.value.ETE.filters).map(v=>v).some(v=> v!=="")){
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.ETE.filteredItems.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.ETE.filteredItems.push(newItem)
              }
            })
            } else {
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.ETE.items.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.ETE.items.push(newItem)
              }
            })
              }
          DigitalEditionsData.value.ETE.facets = newUnits.data.facets
          DigitalEditionsData.value.ETE.pagination['total'] = newUnits.data.total
          DigitalEditionsData.value.ETE.page++
          DigitalEditionsData.value.ETE.loading = false
          })
        },
        isFiltered: () => {
          if(Object.entries(DigitalEditionsData.value.ETE.filters).map(v=>v[0]).some(v=> DigitalEditionsData.value.ETE.filters[v]!=="")){
            return true
          } else {
            return false
          }
        },
      },
      DRE: {
        edition: "DRE",
        title: "Diplomatic Reports Edition",
        description: "The online edition \"Diplomatic Reports\" focuses on how diplomatic staff reported the persecution and murder of European Jews during World War II.",
        pagination:{
          total: null,
        },
        page: 1,
        items: [],
        filtered: false,
        filteredItems: [],
        facets: {},
        filters: {
          Coverage: "",
          Creator: "",
          Subject: "",
          Person: "",
          Organisation: ""
        },
        loading: false,
        getUnitsOnScroll: () => {
          DigitalEditionsData.value.DRE.loading = true;
          fetchDRitems(searchTerm.value, DigitalEditionsData.value.DRE.page, 10, DigitalEditionsData.value.DRE.filters)
          .then((newUnits)=>{
            if(Object.values(DigitalEditionsData.value.DRE.filters).map(v=>v).some(v=> v!=="")){
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.DRE.filteredItems.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.DRE.filteredItems.push(newItem)
              }
            })
            } else {
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.DRE.items.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.DRE.items.push(newItem)
              }
            })
              }
          DigitalEditionsData.value.DRE.facets = newUnits.data.facets
          DigitalEditionsData.value.DRE.pagination['total'] = newUnits.data.total
          DigitalEditionsData.value.DRE.page++
          DigitalEditionsData.value.DRE.loading = false
          })
        },
        isFiltered: () => {
          if(Object.entries(DigitalEditionsData.value.DRE.filters).map(v=>v[0]).some(v=> DigitalEditionsData.value.DRE.filters[v]!=="")){
            return true
          } else {
            return false
          }
        }
      },
      BGFE: {
        edition: "BGFE",
        title: "BeGrenzte Flucht Edition",
        description: "The Austrian refugees on the border with Czechoslovakia in the crisis year 1938. (Edition available in German)",
        pagination:{

          total: null,
        },
        page: 1,
        items: [],
        filtered: false,
        filteredItems: [],
        facets: {},
        filters: {
          Type: "",
          Creator: "",
          Subject: "",
          Person: "",
          Organisation: "",
          Place: ""
        },
        loading: false,
        getUnitsOnScroll: () => {
          DigitalEditionsData.value.BGFE.loading = true;
          fetchBGFitems(searchTerm.value, DigitalEditionsData.value.BGFE.page, 10, DigitalEditionsData.value.BGFE.filters)
          .then((newUnits)=>{
            if(Object.values(DigitalEditionsData.value.BGFE.filters).map(v=>v).some(v=> v!=="")){
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.BGFE.filteredItems.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.BGFE.filteredItems.push(newItem)
              }
            })
            } else {
              newUnits.data.records.forEach(newItem => {
              if (!DigitalEditionsData.value.BGFE.items.some(item => item.id === newItem.id)) {
                DigitalEditionsData.value.BGFE.items.push(newItem)
              }
            })
              }
          DigitalEditionsData.value.BGFE.facets = newUnits.data.facets
          DigitalEditionsData.value.BGFE.pagination['total'] = newUnits.data.total
          DigitalEditionsData.value.BGFE.page++
          DigitalEditionsData.value.BGFE.loading = false
          })
        },
        isFiltered: () => {
          if(Object.entries(DigitalEditionsData.value.BGFE.filters).map(v=>v[0]).some(v=> DigitalEditionsData.value.BGFE.filters[v]!=="")){
            return true
          } else {
            return false
          }
        },
      },
      })

    const handleFilter = (val, type, edition) => {
      if (type ==='removeAll'){
        Object.keys(DigitalEditionsData.value[edition].filters).forEach(
            k=> {
              DigitalEditionsData.value[edition].filters[k]=''
            }
        )
        DigitalEditionsData.value[edition].page = 1
        DigitalEditionsData.value[edition].items = []
        DigitalEditionsData.value[edition].filteredItems = []
        filterKey.value+=1
        DigitalEditionsData.value[edition].getUnitsOnScroll()
      }
      else if(val.length && val!==type) {
        DigitalEditionsData.value[edition].page = 1
        DigitalEditionsData.value[edition].items = []
        DigitalEditionsData.value[edition].filteredItems = []
        DigitalEditionsData.value[edition].filters[type]=val
        filterKey.value+=1
        DigitalEditionsData.value[edition].getUnitsOnScroll()
      }
      else if(!val.length && val!==type) {
        DigitalEditionsData.value[edition].filters[type]=""
        DigitalEditionsData.value[edition].page = 1
        DigitalEditionsData.value[edition].items = []
        DigitalEditionsData.value[edition].filteredItems = []
        filterKey.value+=1
        DigitalEditionsData.value[edition].getUnitsOnScroll()
      }
      chartRerender()
    }

    const selectEdition = (key)=>{
      selectedEdition.value = {}
      selectedEdition.value.page = 1
      selectedEdition.value = DigitalEditionsData.value[key.target.value]
      selectedEdition.value.getUnitsOnScroll()
      chartRerender()
    }

    const chartRerender = () => {
      chartKey.value+=1
    }

    const filterBarClass = computed(() => {
            return showFilterBar.value
            ? "w-full h-max m-0 p-0 bg-ehri-purple lg:w-auto lg:block transition-all ease-in-out duration-600"
            : "w-full h-0 transition-all ease-in-out overflow-hidden lg:overflow-scroll duration-800 lg:w-auto ";
        });

   
    const toggleFilterBar = () => {
      showFilterBar.value = !showFilterBar.value;
    };

    watch(searchTerm, () => {
      DigitalEditionsData.value.ETE.getUnitsOnScroll()
      DigitalEditionsData.value.DRE.getUnitsOnScroll()
      DigitalEditionsData.value.BGFE.getUnitsOnScroll()
    });

    const loading = ref(true)

    watch([DigitalEditionsData.value], ()=>{
      if (Object.values(DigitalEditionsData.value).map(v => v).every(v => !v.loading)
          && (!selectedEdition.value || (selectedEdition.value && !selectedEdition.value['isFiltered']))
      ){
        loading.value = true
        let entries = Object.entries(DigitalEditionsData.value).map(v => v);
        sortedDEResults.value = entries.sort((a, b) => b[1]['pagination']['total'] - a[1]['pagination']['total'])
        sortedDEResultsNonNull.value = sortedDEResults.value.filter(i => {
              if (i[1]['pagination']['total']>0){
                return i
              }
            }
        )
        if(selectedEdition.value['items']&&selectedEdition.value['items'].length){
          return false
        } else {
          selectedEdition.value = DigitalEditionsData.value[sortedDEResultsNonNull.value[0][0]]
        }
        loading.value = false
      }
    })

    useInfiniteScroll(
      el,
      async () => {
        await selectedEdition.value.getUnitsOnScroll()
      },
      { distance: 300 }
    )
    DigitalEditionsData.value.ETE.getUnitsOnScroll()
    DigitalEditionsData.value.DRE.getUnitsOnScroll()
    DigitalEditionsData.value.BGFE.getUnitsOnScroll()

    return {
      loading,
      sortedDEResultsNonNull,
      selectEdition,
      selectedEdition,
      handleFilter,
      chartKey,
      el,
      filterKey,
      showFilterBar,
      toggleFilterBar,
      filterBarClass
    };
  },
};
</script>

<style scoped>

</style>
