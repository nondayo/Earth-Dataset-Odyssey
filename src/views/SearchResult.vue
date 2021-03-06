<template>
  <div>
    <!-- empty result -->
    <div v-if="emptyRes" class="w-full">
      <div class="wrap w-full">
        <div
          class="header flex justify-between items-center w-4/5 mb-6 mx-auto"
        >
          <a href="javascript:void(0);" class="text-24R font-bold">
            Datasets of
            <span class="text-lightgreen"> {{ searchKeyWord }} (0)</span>
          </a>
        </div>
        <img src="../assets/image/noMatchResult.png" alt="" class="w-full" />
        <div class="content flex flex-col items-center text-white">
          <h2 class="text-5.5xl font-bold">No Results</h2>
          <h4 class="text-2xl font-normal">No Datasets found</h4>

          <router-link
            :to="{ name: 'RecommendView' }"
            class="p-4 mt-6 bg-lightgreen rounded-2rem text-white text-2xl font-semibold text-center"
          >
            Back to Earch
          </router-link>
        </div>
      </div>
    </div>
    <!-- main result -->
    <div v-else class="main flex flex-col mx-auto w-4/5">
      <div class="header flex justify-between items-center">
        <a href="javascript:void(0);" class="text-24R font-bold">
          Datasets of
          <span class="text-lightgreen"
            >{{ searchKeyWord }} ({{ relatedDataList.length }})</span
          >
        </a>
      </div>
      <div
        class="flex my-10 cursor-pointer"
        @click="toDataPage(eachdata.id)"
        v-for="(eachdata, index) in relatedDataList"
        :key="eachdata.id + '_' + index"
      >
        <div class="mr-10 rounded-1rem">
          <img
            :src="baseImageURL + eachdata.id + '?h=200&w=250'"
            alt=""
            width="250"
          />
        </div>
        <div class="w-full">
          <h3 class="mb-6 text-3xl font-bold">
            {{ eachdata.title }}
          </h3>
          <div class="info-key w-full min-w-30vw text-3xl font-semibold">
            <table class="w-full">
              <colgroup>
                <col span="1" style="width: 20%" />
                <col span="1" style="width: 80%" />
              </colgroup>
              <tr class=" ">
                <td>Summary</td>
                <td class="text-2xl font-normal break-normal text-justify">
                  {{ eachdata.cmr.summary }}
                </td>
              </tr>
              <tr class=" ">
                <td>Temporal Coverage</td>
                <td class="text-2xl font-normal">
                  {{
                    data_format(eachdata.time_start, true) +
                    " to " +
                    data_format(eachdata.time_end, true)
                  }}
                </td>
              </tr>
              <tr class="leading-12">
                <td>Data Format</td>
                <td class="text-2xl font-normal">
                  {{ eachdata.original_format }}
                </td>
              </tr>
              <tr class="leading-12">
                <td>Number of Granules</td>
                <td class="text-2xl font-normal">
                  {{ eachdata.granule_count }}
                </td>
              </tr>
              <tr class="">
                <td>URL</td>
                <td class="text-2xl font-normal">
                  <a
                    class="border rounded-lg p-2"
                    :href="dataDoc + eachdata.id + '.html'"
                    target="_blank"
                    >{{ dataDoc + eachdata.id + ".html" }}</a
                  >
                </td>
              </tr>
              <tr class="leading-12">
                <td>Totla of Votes</td>
                <td class="flex text-2xl font-normal">
                  <div class="mr-8">
                    <span class="text-lightgreen mr-2">4</span>Researches
                  </div>
                  <div class="mr-8">
                    <span class="text-lightgreen mr-2">10</span>Analysis
                  </div>
                  <div class="mr-8">
                    <span class="text-lightgreen mr-2">10</span>citizen
                    scientist
                  </div>
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex"

export default {
  name: "SearchResult",
  computed: {
    ...mapGetters({
      searchKeyWord: "get_searchKeyWord",
      baseImageURL: "getBaseImageURL",
      dataDoc: "get_dataDoc",
    }),
  },
  data() {
    return {
      relatedDataList: [],
      emptyRes: false,
    }
  },
  watch: {
    searchKeyWord: {
      handler() {
        this.$store
          .dispatch("search_api")
          .then((res) => {
            this.relatedDataList = res
          })
          .catch(() => {
            this.emptyRes = true
          })
      },
    },
  },
  mounted() {
    if (this.searchKeyWord.length === 0) {
      this.$router.push({ name: "RecommendView" })
    }
    this.$store
      .dispatch("search_api")
      .then((res) => {
        this.relatedDataList = res
      })
      .catch(() => {
        this.emptyRes = true
      })
  },
  methods: {
    toDataPage(dataset_id) {
      if (this.$route.name !== "DataInfo")
        this.$router.push({ name: "DataInfo", params: { dataId: dataset_id } })
      this.$store.dispatch(
        "set_dataDetailInfo",
        this.relatedDataList.find((el) => el.id === dataset_id)
      )
      this.pushNewRecode_view(dataset_id)

      window.scrollTo(0, 0)
    },
    data_format(raw_date, to_day = false) {
      if (to_day) return new Date(raw_date).format("yyyy-MM-dd")
      return new Date(raw_date).format("yyyy-MM-dd hh:mm:ss")
    },
    pushNewRecode_view(dataset_id) {
      const newRecodeObj = {
        id: dataset_id,
        time: new Date().format("yyyy-MM-dd hh:mm:ss"),
      }
      this.$store.dispatch("push_new_recode", {
        recodeType: "view",
        newRecodeObj,
      })
    },
  },
}
</script>

<style scoped>
.empty-container {
  background-image: url("../assets/image/noMatchResult.png");
  background-size: contain;
}
.wrap {
  display: inline-block;
  position: relative;
}

.wrap img + .content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
