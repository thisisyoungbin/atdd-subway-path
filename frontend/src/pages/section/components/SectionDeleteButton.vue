<template>
  <v-btn @click="onDeleteLine" icon>
    <v-icon color="grey lighten-1">mdi-delete</v-icon>
  </v-btn>
</template>

<script>
import { mapMutations } from "vuex";
import { SET_LINE, SHOW_SNACKBAR } from "../../../store/shared/mutationTypes";
import { SNACKBAR_MESSAGES } from "../../../utils/constants";

export default {
  name: "SectionDeleteButton",
  props: {
    lineId: {
      type: String,
      required: true,
    },
    stationId: {
      type: String,
      required: true,
    },
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_LINE]),
    async onDeleteLine() {
      try {
        const delete_response = await fetch(`http://localhost:8080/lines/${this.lineId}/sections?stationId=${this.stationId}`, {
          method: "DELETE"
        });

        if(!delete_response.ok){
          throw new Error(`${delete_response.status}`);
        }

        const line_response = await fetch(`http://localhost:8080/lines/${this.lineId}`)
        if (!line_response.ok) {
          throw new Error(`${line_response.status}`);
        }

        const line = await line_response.json();
        this.setLine({ ...line })
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.SUCCESS);
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.COMMON.FAIL);
        throw new Error(e);
      }
    },
  },
};
</script>
