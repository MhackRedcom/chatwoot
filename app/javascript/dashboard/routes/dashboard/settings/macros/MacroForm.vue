<template>
  <div class="row">
    <div class="small-8 columns with-right-space macros-canvas">
      <macro-nodes
        v-model="macro.actions"
        @addNewNode="appendNode"
        @deleteNode="deleteNode"
        @resetAction="resetNode"
      />
    </div>
    <div class="small-4 columns">
      <macro-properties
        :macro-name="macro.name"
        :macro-visibility="macro.visibility"
        @update:name="updateName"
        @update:visibility="updateVisibility"
        @submit="submit"
      />
    </div>
  </div>
</template>

<script>
import MacroNodes from './MacroNodes';
import MacroProperties from './MacroProperties';
import { required, requiredIf } from 'vuelidate/lib/validators';

export default {
  components: {
    MacroNodes,
    MacroProperties,
  },
  provide() {
    return {
      $v: this.$v,
    };
  },
  props: {
    macroData: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      macro: this.macroData,
    };
  },
  watch: {
    macroData: {
      handler() {
        this.macro = this.macroData;
      },
      immediate: true,
    },
  },
  validations: {
    macro: {
      name: {
        required,
      },
      visibility: {
        required,
      },
      actions: {
        required,
        $each: {
          action_params: {
            required: requiredIf(prop => {
              if (prop.action_name === 'send_email_to_team') return true;
              return !(
                prop.action_name === 'mute_conversation' ||
                prop.action_name === 'snooze_conversation' ||
                prop.action_name === 'resolve_conversation'
              );
            }),
          },
        },
      },
    },
  },
  mounted() {
    this.$v.$reset();
  },
  methods: {
    updateName(value) {
      this.macro.name = value;
    },
    updateVisibility(value) {
      this.macro.visibility = value;
    },
    appendNode() {
      this.macro.actions.push({
        action_name: 'assign_team',
        action_params: [],
      });
    },
    deleteNode(index) {
      this.macro.actions.splice(index, 1);
    },
    submit() {
      this.$v.$touch();
      if (this.$v.$invalid) return;
      this.$emit('submit', this.macro);
    },
    resetNode(index) {
      this.macro.actions[index].action_params = [];
    },
  },
};
</script>

<style scoped lang="scss">
.row {
  height: 100%;
}
.macros-canvas {
  background-image: radial-gradient(var(--s-100) 1.2px, transparent 0);
  background-size: var(--space-normal) var(--space-normal);
  height: 100%;
  max-height: 100%;
  padding: var(--space-normal) var(--space-three);
  max-height: 100vh;
  overflow-y: auto;
}
</style>
