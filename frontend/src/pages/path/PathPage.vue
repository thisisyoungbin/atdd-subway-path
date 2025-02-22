<template>
  <v-sheet class="d-flex flex-column justify-center mt-12">
    <div class="d-flex justify-center relative">
      <v-card width="500" max-width="600" max-height="600" class="card-border">
        <v-card-title class="font-weight-bold justify-center relative">
          경로 검색
        </v-card-title>
        <v-card-text
          class="relative mt-2 px-0 line-list-container d-flex flex-column"
        >
          <div class="px-4 pb-6">
            <div class="d-flex width-100">
              <v-select
                v-model="path.source"
                class="pr-4 path-station-select"
                :items="allStationsView"
                label="출발역"
                color="grey darken-1"
                item-color="amber darken-3"
                outlined
                dense
              ></v-select>
              <v-icon class="relative arrow-right-icon"
                >mdi-arrow-right-bold</v-icon
              >
              <v-select
                v-model="path.target"
                class="pl-4 path-station-select"
                :items="allStationsView"
                label="도착역"
                color="grey darken-1"
                item-color="amber darken-3"
                outlined
                dense
              ></v-select>
            </div>
            <div class="d-flex mb-4">
              <v-btn
                @click="onSearchResult"
                color="amber"
                class="w-100"
                depressed
                >검색</v-btn
              >
            </div>
            <template v-if="pathResult">
              <v-divider />
              <div class="d-flex justify-center mt-4">
                <v-card width="400" flat>
                  <v-tabs
                    v-model="tab"
                    background-color="transparent"
                    color="amber"
                    grow
                  >
                    <v-tab>최단 거리</v-tab>
                    <v-tab @click="onSearchMinimumDurationType"
                      >최소 시간</v-tab
                    >
                  </v-tabs>
                  <v-tabs-items v-if="pathResult" v-model="tab">
                    <v-tab-item>
                      <v-simple-table>
                        <template v-slot:default>
                          <thead>
                            <tr>
                              <th class="text-center">거리</th>
                              <th class="text-center">요금</th>
                            </tr>
                          </thead>
                          <tbody class="text-center">
                            <tr>
                              <td>{{ pathResult.distance }}km</td>
                              <td>{{ pathResult.fare }}원</td>
                            </tr>
                          </tbody>
                        </template>
                      </v-simple-table>
                    </v-tab-item>
                    <v-tab-item v-if="pathResultByMinimumDuration">
                      <v-simple-table>
                        <template v-slot:default>
                          <thead>
                            <tr>
                              <th class="text-center">시간</th>
                              <th class="text-center">요금</th>
                            </tr>
                          </thead>
                          <tbody class="text-center">
                            <tr>
                              <td>
                                {{ pathResultByMinimumDuration.duration }}분
                              </td>
                              <td>{{ pathResultByMinimumDuration.fare }}원</td>
                            </tr>
                          </tbody>
                        </template>
                      </v-simple-table>
                    </v-tab-item>
                  </v-tabs-items>
                </v-card>
              </div>
              <v-divider />
              <div class="d-flex justify-center mt-4">
                <v-card width="400" flat>
                  <template v-for="(station, index) in pathResult.stations">
                    <span :key="station.id">
                      <v-chip
                        :key="index"
                        class="ma-2"
                        :color="
                          index === 0 ||
                          index === pathResult.stations.length - 1
                            ? 'amber'
                            : ''
                        "
                      >
                        <v-avatar
                          v-if="
                            index === 0 ||
                            index === pathResult.stations.length - 1
                          "
                          left
                        >
                          <v-icon>mdi-subway</v-icon>
                        </v-avatar>
                        {{ station.name }}
                      </v-chip>
                      <v-icon v-if="index < pathResult.stations.length - 1"
                        >mdi-arrow-right-bold</v-icon
                      >
                    </span>
                  </template>
                </v-card>
              </div>
            </template>
          </div>
        </v-card-text>
      </v-card>
    </div>
  </v-sheet>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import { SET_STATIONS, SHOW_SNACKBAR } from "../../store/shared/mutationTypes";
import { SNACKBAR_MESSAGES } from "../../utils/constants";
import validator from "../../utils/validator";

export default {
  name: "PathPage",
  computed: {
    ...mapGetters(["stations"]),
  },
  created() {
    this.initAllStationsView();
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_STATIONS]),
    async onSearchResult() {
      try {
        const path_response = await fetch(`http://localhost:8080/paths?source=${this.path.source}&target=${this.path.target}`)
        if (!path_response.ok) {
          throw new Error(`${path_response.status}`);
        }
        this.pathResult = await path_response.json();
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
    async initAllStationsView() {
      try {
        const stations_response = await fetch("http://localhost:8080/stations")
        if (!stations_response.ok) {
          throw new Error(`${stations_response.status}`);
        }
        const stations = await stations_response.json();
        this.setStations(stations)
        if (this.stations.length < 1) {
          return;
        }
        this.allStationsView = this.stations.map((station) => {
          return {
            text: station.name,
            value: station.id,
          };
        });
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
    async onSearchMinimumDurationType() {
      try {
        // TODO 최소 시간을 검색하는 API를 추가해주세요.
        // this.pathResultByMinimumDuration = await fetch("/paths", {})
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
  },
  data() {
    return {
      path: {
        source: "",
        target: "",
      },
      pathResult: null,
      pathResultByMinimumDuration: null,
      allStationsView: [],
      rules: { ...validator },
      tab: null,
    };
  },
};
</script>
<style scoped>
.path-station-select {
  width: 200px;
}

.arrow-right-icon {
  bottom: 15px;
}
</style>
