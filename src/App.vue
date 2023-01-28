<template>
  <div class="xl:p-0 p-10 h-screen max-w-screen-2xl mx-auto">
    <div class="mb-auto xl:m-5">
      <div class="sm:flex sm:items-center sm:justify-between my-5">
        <div
          @click="reloadPage"
          class="font-bold text-2xl text-white cursor-pointer"
        >
          <img class="h-10" src="/Netflix_logo.svg" alt="Netflix logo" />
        </div>
        <div class="flex items-center">
          <h1
            class="text-white font-bold mr-5 animate-pulse"
            v-if="(results.length == 0) & (searchInput.search != '')"
          >
            Need a more specific search, try again...
          </h1>
          <h1
            v-else-if="searchInput.search != ''"
            class="text-white font-bold mr-5"
          >
            {{ results.length }} results for "{{ searchInput.search }}"
          </h1>
          <div class="w-70 sm:m-0 mt-5">
            <div>
              <div class="flex bg-white rounded-2 px-5 py-2">
                <input
                  type="text"
                  v-model="searchInput.search"
                  placeholder="search with Amazon..."
                  class="focus:outline-none w-full"
                />
                <button class="text-black font-bold" @click="fetchUrl">
                  <Search class="h-5 text-netflix" />
                </button>
              </div>
              <!-- <div
                class="z-10 absolute w-70 bg-grey divide-y divide-dark-800 rounded"
              >
                <div v-if="searchInput.search != ''">
                  <ul
                    class="py-1 text-sm text-white"
                    v-if="searchSuggestions.length > 0"
                  >
                    <li v-for="s of searchSuggestions" :key="s">
                      <a
                        href="#"
                        @click="searchInput.search = s"
                        class="block font-medium px-4 py-2 hover:text-netflix"
                        >{{ s }}</a
                      >
                    </li>
                  </ul>
                </div>
              </div> -->
            </div>
          </div>
        </div>
      </div>
      <div class="sm:flex justify-between">
        <div class="w-60 mr-5 p-5 h-min rounded-2 bg-dark-800 mb-5 sm:mb-0">
          <h1 class="text-bordeaux font-bold text-xl mb-5 flex items-center">
            <Filter class="mr-1" /> Filter
          </h1>
          <h2 class="font-bold mb-1 text-netflix">TYPES</h2>
          <div class="flex items-center">
            <input
              type="checkbox"
              v-model="filterInputType.TV"
              class="
                cursor-pointer
                w-4
                h-4
                bg-grey
                rounded
                focus:ring-netflix
                hover:ring-netflix
                my-2
              "
            />
            <label class="ml-2 text-sm font-medium text-white">TV show</label>
          </div>
          <div class="flex items-center">
            <input
              type="checkbox"
              v-model="filterInputType.Movie"
              class="
                cursor-pointer
                w-4
                h-4
                bg-grey
                rounded
                focus:ring-netflix
                hover:ring-netflix
                my-2
              "
            />
            <label class="ml-2 text-sm font-medium text-white">Movie</label>
          </div>
          <h2 class="text-netflix font-bold mb-1 mt-5">SORT ON RELEASE</h2>
          <div class="flex text-white font-medium space-x-5">
            <div
              class="flex cursor-pointer"
              @click="
                (dropdownText = 'Select Director'),
                  fetchSort('asc'),
                  (searchInput.search = '')
              "
            >
              <div class="flex items-center"><SortAsc /> ASC</div>
            </div>
            <div
              class="flex cursor-pointer"
              @click="
                (dropdownText = 'Select Director'),
                  fetchSort('desc'),
                  (searchInput.search = '')
              "
            >
              <div class="flex items-center"><SortDesc /> DESC</div>
            </div>
          </div>
          <h2 class="text-netflix font-bold mb-1 mt-5">DIRECTOR</h2>
          <div class="relative">
            <button
              class="
                flex
                items-center
                justify-around
                bg-dark-400
                w-48
                text-white
                py-2
                rounded-2
                font-medium
              "
              @click="showDropdown = !showDropdown"
            >
              {{ dropdownText }}
              <ChevronDown class="h-5" v-if="showDropdown != true" />
              <ChevronUp class="h-5" v-else />
            </button>
            <div
              v-if="showDropdown"
              class="
                absolute
                z-10
                h-50
                mt-2
                py-3
                w-48
                bg-dark-400
                rounded-md
                shadow-md
                overflow-y-auto
              "
            >
              <a
                v-if="dropdownText != 'Select Director'"
                @click="
                  (dropdownText = 'Select Director'),
                    (showDropdown = false),
                    fetchUrl()
                "
                href="#"
                class="
                  block
                  px-4
                  py-3
                  text-sm text-red-500
                  hover:text-bordeaux
                  font-medium
                  border-b-1 border-dark-200
                "
              >
                Remove selection
              </a>
              <a
                @click="
                  (showDropdown = false),
                    (searchInput.search = ''),
                    (filterInputType.TV = false),
                    (filterInputType.Movie = false),
                    (dropdownText = d.director),
                    fetchDirector(d.director)
                "
                href="#"
                v-for="d of directors"
                :key="d"
                class="
                  block
                  px-4
                  py-2
                  text-sm text-white
                  hover:text-bordeaux
                  font-medium
                "
              >
                {{ d.director }}
              </a>
            </div>
          </div>
        </div>

        <div
          class="grid grid-cols-1 xl:grid-cols-3 md:grid-cols-2 gap-10 mb-10"
          v-if="results.length > 0"
        >
          <div
            v-for="r of results"
            :key="r"
            class="
              max-w-sm
              min-w-55
              p-6
              bg-dark-800
              rounded-lg
              shadow-md
              relative
            "
          >
            <div class="flex justify-between items-center">
              <div class="flex items-center text-white">
                <Tv v-if="r.type == 'TV Show'" class="my-2 mr-1" />
                <Clapperboard v-else class="my-2 mr-1" />
                <p class="align-baseline">{{ r.type }}</p>
              </div>
              <p class="text-grey font-bold">{{ r.release_year }}</p>
            </div>

            <h5 class="mb-2 text-2xl font-semibold tracking-tight text-netflix">
              {{ r.title }}
            </h5>

            <p class="mb-3 font-normal text-light-900">{{ r.description }}</p>
            <div class="mt-5">
              <h1 class="font-bold text-white">CATEGORY</h1>
              <p class="font-medium text-bordeaux">{{ r.listed_in }}</p>
            </div>
            <div class="mt-2 flex text-light-900 font-medium items-center">
              <Clock class="mr-1 h-5" /> {{ r.duration }}
            </div>
            <div class="text-grey float-right my-5" v-if="r.director != ''">
              <p
                class="absolute bottom-0 right-0 p-5 font-bold"
                v-if="r.director.length < 30"
              >
                <span class="font-medium"> Made by </span>
                {{ r.director }}
              </p>
            </div>
            <div class="text-grey float-right my-5" v-else>
              <p class="absolute bottom-0 right-0 p-5 font-bold">
                <span class="font-medium"> No director found </span>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, watch } from "vue";
import {
  Clapperboard,
  Tv,
  Search,
  Filter,
  SortAsc,
  SortDesc,
  Clock,
  ChevronDown,
  ChevronUp,
} from "lucide-vue-next";
export default {
  components: {
    Clapperboard,
    Tv,
    Search,
    Filter,
    SortAsc,
    SortDesc,
    Clock,
    ChevronDown,
    ChevronUp,
  },
  setup() {
    const host = import.meta.env.VITE_OPENSEARCH_ENDPOINT;

    let results = ref([]);
    let data = {};
    let showDropdown = ref(false);
    let dropdownText = ref("Select Director");
    let directors = ref([]);

    const fetchTitle = async () => {
      // GET request using fetch with async/await
      const response = await fetch(`${host}q=${searchInput.search}`);
      data = {};
      results.value = [];
      data = await response.json();
      for await (let item of data.hits.hits) {
        results.value.push(item._source);
      }
    };

    const fetchDirector = async (d) => {
      // GET request using fetch with async/await
      const response = await fetch(`${host}d=${d}`);
      data = {};
      results.value = [];
      data = await response.json();
      for await (let item of data.hits.hits) {
        results.value.push(item._source);
      }
    };

    const fetchSort = async (s) => {
      // GET request using fetch with async/await
      const response = await fetch(`${host}sort=release_year:${s}`);
      data = {};
      results.value = [];
      data = await response.json();
      for await (let item of data.hits.hits) {
        results.value.push(item._source);
      }
    };

    const fetchType = async (t) => {
      // GET request using fetch with async/await
      const response = await fetch(`${host}t=${t}`);
      data = {};
      results.value = [];
      data = await response.json();
      for await (let item of data.hits.hits) {
        results.value.push(item._source);
      }
    };

    const fetchUrl = async () => {
      // GET request using fetch with async/await
      const response = await fetch(`${host}_search`);
      data = {};
      results.value = [];
      directors.value = [];
      data = await response.json();
      for await (let item of data.hits.hits) {
        results.value.push(item._source);
        if (item._source.director != "") {
          directors.value.push(item._source);
        }
      }
    };

    const searchInput = reactive({
      search: "",
    });

    const filterInputType = reactive({
      Movie: false,
      TV: false,
    });

    const reloadPage = () => {
      window.location.reload();
    };

    // Watchers
    watch(searchInput, () => {
      dropdownText.value = "Select Director";
      if (searchInput.search == "") {
        fetchUrl();
      }
      // sort.value = ''
      fetchTitle();
    });

    watch(filterInputType, () => {
      dropdownText.value = "Select Director";
      searchInput.search = "";
      if (filterInputType.Movie == true && filterInputType.TV == true) {
        fetchUrl();
      }
      if (filterInputType.Movie == false && filterInputType.TV == false) {
        fetchUrl();
      }
      if (filterInputType.Movie == true) {
        fetchType("Movie");
      }
      if (filterInputType.TV == true) {
        fetchType("TV Show");
      }
    });

    fetchUrl();

    return {
      results,
      searchInput,
      dropdownText,
      showDropdown,
      filterInputType,
      directors,

      fetchTitle,
      fetchUrl,
      fetchDirector,
      fetchType,
      fetchSort,

      reloadPage,
    };
  },
};
</script>
