<template>
  <Dialog :close="close">
    <template slot="trigger">
      <v-btn
        @click="initLineView"
        class="mx-2 section-create-button"
        fab
        color="amber"
        depressed
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </template>
    <template slot="title">
      <div class="width-100 text-center mt-6">구간 추가</div>
    </template>
    <template slot="text">
      <v-form ref="sectionForm" v-model="valid" @submit.prevent>
        <v-select
          v-model="sectionForm.lineId"
          :items="lineNameViews"
          @change="onChangeLine"
          label="노선 선택"
          width="400"
          item-color="amber darken-3"
          color="grey darken-1"
          outlined
          dense
        ></v-select>
        <div class="d-flex">
          <v-select
            v-model="sectionForm.upStationId"
            class="pr-5"
            :items="allStationsView"
            label="상행역"
            width="400"
            color="grey darken-1"
            item-color="amber darken-3"
            outlined
            dense
          ></v-select>
          <v-select
            v-model="sectionForm.downStationId"
            class="pl-5"
            :items="allStationsView"
            label="하행역"
            width="400"
            color="grey darken-1"
            item-color="amber darken-3"
            outlined
            dense
          ></v-select>
        </div>
        <div class="d-flex">
          <v-text-field
            v-model="sectionForm.distance"
            :rules="rules.section.distance"
            color="grey darken-1"
            label="거리"
            placeholder="거리"
            outlined
          ></v-text-field>
        </div>
      </v-form>
    </template>
    <template slot="action">
      <v-btn
        :disabled="!valid"
        @click.prevent="onCreateSection"
        color="amber"
        depressed
        >확인</v-btn
      >
    </template>
  </Dialog>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import dialog from "../../../mixins/dialog";
import Dialog from "../../../components/dialogs/Dialog";
import {
  SET_LINE,
  SET_LINES,
  SHOW_SNACKBAR,
} from "../../../store/shared/mutationTypes";
import { SNACKBAR_MESSAGES } from "../../../utils/constants";
import validator from "../../../utils/validator";

export default {
  name: "SectionCreateButton",
  components: { Dialog },
  mixins: [dialog],
  computed: {
    ...mapGetters(["lines", "stations"]),
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_LINES, SET_LINE]),
    initLineView() {
      if (this.lines.length < 1) {
        return;
      }
      this.lineNameViews = this.lines.map(({ name, id }) => {
        return {
          text: name,
          value: id,
        };
      });
    },
    async initLineStationsView() {
      try {
        const selected_line_response =  await fetch(`http://localhost:8080/lines/${this.sectionForm.lineId}`);
        if (!selected_line_response.ok) {
          throw new Error(`${selected_line_response.status}`);
        }
        this.activeLine = await selected_line_response.json();

        if (this.selectedLine.stations?.length < 1) {
          return;
        }
        this.lineStationsNameViews = this.selectedLine.stations?.map(
          (station) => {
            return {
              text: station.name,
              value: station.id,
            };
          }
        );
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
    async initAllStationsView() {
      try {
        if (this.stations.length < 1) {
          return;
        }
        this.allStationsView = this.stations.map(({ name, id }) => {
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
    isValid() {
      return this.$refs.sectionForm.validate();
    },
    onChangeLine() {
      this.initLineStationsView();
      this.initAllStationsView();
    },
    async onCreateSection() {
      if (!this.isValid()) {
        return;
      }
      try {
        const addSectionRequest = {
          "upStationId": this.sectionForm.upStationId,
          "downStationId": this.sectionForm.downStationId,
          "distance": this.sectionForm.distance
        }
        const add_section_response = await fetch(`http://localhost:8080/lines/${this.sectionForm.lineId}/sections`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(addSectionRequest)
        });
        if (!add_section_response.ok) {
          throw new Error(`${add_section_response.status}`);
        }

        const lines_response = await fetch("http://localhost:8080/lines");

        if (!lines_response.ok) {
          throw new Error(`${lines_response.status}`);
        }

        const lines = await lines_response.json();

        this.setLines(lines)
        const line = this.lines.find(({ id }) => id === this.selectedLine.id);
        this.setLine(line);
        this.$refs.sectionForm.resetValidation();
        this.initSectionForm();
        this.closeDialog();
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.SUCCESS);
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
    initSectionForm() {
      this.sectionForm = {
        lineId: "",
        upStationId: "",
        downStationId: "",
        distance: "",
      };
    },
  },
  data() {
    return {
      rules: { ...validator },
      sectionForm: {
        lineId: "",
        upStationId: "",
        downStationId: "",
        distance: "",
      },
      selectedLine: {},
      lineStationsNameViews: [],
      allStationsView: [],
      lineNameViews: [],
      valid: false,
    };
  },
};
</script>

<style lange="scss" scoped>
.section-create-button {
  top: -29px;
}
</style>
