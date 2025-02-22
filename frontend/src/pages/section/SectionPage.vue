<template>
  <v-sheet class="d-flex flex-column justify-center mt-12">
    <div class="d-flex justify-center relative">
      <v-card width="500" max-width="600" max-height="600" class="card-border">
        <v-card-title class="font-weight-bold justify-center relative">
          구간 관리
        </v-card-title>
        <v-card-text
          class="relative px-0 line-list-container d-flex flex-column"
        >
          <div class="relative px-5 pb-5">
            <v-select
              v-model="activeLineId"
              :items="lineNamesViews"
              @change="onChangeLine"
              label="노선 선택"
              width="400"
              color="grey darken-1"
              item-color="amber darken-3"
              outlined
              dense
            ></v-select>
          </div>
          <v-divider />
          <div class="d-flex justify-end mr-4 section-create-button-container">
            <SectionCreateButton />
          </div>
          <div class="mt-10 overflow-y-auto">
            <v-card v-if="activeLine.id" class="mx-5" outlined>
              <v-toolbar :color="activeLine.color" dense flat>
                <v-toolbar-title>{{ activeLine.name }}</v-toolbar-title>
              </v-toolbar>
              <v-card-text class="overflow-y-auto py-0">
                <v-list dense class="max-height-300px">
                  <template
                    v-if="activeLine.stations && activeLine.stations.length > 0"
                  >
                    <v-list-item
                      v-for="(station, index) in activeLine.stations"
                      :key="index"
                    >
                      <v-list-item-content>
                        <v-list-item-title
                          v-text="station.name"
                        ></v-list-item-title>
                      </v-list-item-content>
                      <v-list-item-action class="flex-row">
                        <SectionDeleteButton
                          :line-id="activeLine.id"
                          :station-id="station.id"
                        />
                      </v-list-item-action>
                    </v-list-item>
                  </template>
                  <template v-else>
                    <v-list-item>
                      <v-list-item-content>
                        <v-list-item-title
                          >아직 추가된 역이 없습니다.</v-list-item-title
                        >
                      </v-list-item-content>
                    </v-list-item>
                  </template>
                </v-list>
              </v-card-text>
            </v-card>
          </div>
        </v-card-text>
      </v-card>
    </div>
  </v-sheet>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import {
  SET_LINES,
  SET_STATIONS,
  SHOW_SNACKBAR,
} from "../../store/shared/mutationTypes";
import { SNACKBAR_MESSAGES } from "../../utils/constants";
import SectionCreateButton from "./components/SectionCreateButton";
import SectionDeleteButton from "./components/SectionDeleteButton";

export default {
  name: "SectionPage",
  components: { SectionDeleteButton, SectionCreateButton },
  async created() {
    const stations_response = await fetch("http://localhost:8080/stations")
    if (!stations_response.ok) {
      throw new Error(`${stations_response.status}`);
    }
    const stations = await stations_response.json();
    this.setStations([...stations]);

    const lines_response = await fetch("http://localhost:8080/lines");
    if (!lines_response.ok) {
      throw new Error(`${lines_response.status}`);
    }
    const lines = await lines_response.json();
    this.setLines([...lines]);
    this.initLinesView();
  },
  computed: {
    ...mapGetters(["lines", "line"]),
  },
  watch: {
    line() {
      if (this.activeLine.id === this.line.id) {
        this.activeLine = { ...this.line };
      }
    },
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_STATIONS, SET_LINES]),
    initLinesView() {
      try {
        if (this.lines.length < 1) {
          return;
        }
        this.lineNamesViews = this.lines.map(({ name, id }) => {
          return {
            text: name,
            value: id,
          };
        });
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
    async onChangeLine() {
      try {
        const active_line_response =  await fetch(`http://localhost:8080/lines/${this.activeLineId}`);
        if (!active_line_response.ok) {
          throw new Error(`${active_line_response.status}`);
        }
        this.activeLine = await active_line_response.json();
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
  },
  data() {
    return {
      lineNamesViews: [],
      activeLineId: {},
      activeLine: {},
    };
  },
};
</script>

<style lang="scss" scoped>
.section-create-button-container {
  height: 25px;
}
</style>
