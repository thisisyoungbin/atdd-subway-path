<template>
  <v-btn @click="onDeleteLine" icon>
    <v-icon color="grey lighten-1">mdi-delete</v-icon>
  </v-btn>
</template>

<script>
import {mapMutations} from "vuex";
import {SET_LINES, SHOW_SNACKBAR} from "../../../store/shared/mutationTypes";
import {SNACKBAR_MESSAGES} from "../../../utils/constants";

export default {
  name: "LineDeleteButton",
  props: {
    line: {
      type: Object,
      required: true,
    },
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR, SET_LINES]),
    async onDeleteLine() {
      try {
        const delete_line_response = await fetch(`http://localhost:8080/lines/${this.line.id}`, {
          method: "DELETE"
        });

        if(!delete_line_response.ok){
          throw new Error(`${delete_line_response.status}`);
        }

        const lines_response = await fetch("http://localhost:8080/lines");
        if(!lines_response.ok){
          throw new Error(`${lines_response.status}`);
        }

        const lines = await lines_response.json();
        this.setLines([...lines])
        this.showSnackbar(SNACKBAR_MESSAGES.LINE.DELETE.SUCCESS);
      } catch (e) {
        this.showSnackbar(SNACKBAR_MESSAGES.LINE.DELETE.FAIL);
      }
    },
  },
};
</script>
