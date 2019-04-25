<template>
  <nav v-if="hasLock" class="k-form-buttons" theme="lock">
    <k-view>
      <k-text>
        Page is locked by: {{ lock }}
        (last active {{ lock }})
      </k-text>
      <k-button
        :disabled="true"
        icon="lock"
        class="k-form-button"
      >
        {{ $t("unlock") }}
      </k-button>
    </k-view>
  </nav>
  <nav v-else-if="hasChanges" class="k-form-buttons">
    <k-view>
      <k-button
        :disabled="isDisabled"
        icon="undo"
        class="k-form-button"
        @click="reset"
      >
        {{ $t("revert") }}
      </k-button>
      <k-button
        :disabled="isDisabled"
        icon="check"
        class="k-form-button"
        @click="save"
      >
        {{ $t("save") }}
      </k-button>
    </k-view>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      interval: null
    }
  },
  computed: {
    hasChanges() {
      return this.$store.getters["form/hasChanges"](this.id);
    },
    hasLock() {
      return true;
    },
    hasUnlock() {
      return this.unlock !== null;
    },
    id() {
      return this.$store.state.form.current;
    },
    isDisabled() {
      return this.$store.getters["form/isDisabled"];
    },
    lock() {
      return this.$store.state.form.lock;
    },
    unlock() {
      return this.$store.state.form.unlock;
    },
  },
  watch: {
    hasChanges(changes) {
      if (changes === true) {
        this.locking();
        this.interval = setInterval(this.locking, 60 * 1000);
      } else {
        clearInterval(this.interval);
      }
    }
  },
  created() {
    this.$events.$on("keydown.cmd.s", this.save);
  },
  destroyed() {
    this.$events.$off("keydown.cmd.s", this.save);
  },
  methods: {
    locking() {
      this.$api.patch(this.$route.path + "/lock").catch(error => {

      });
    },
    reset() {
      this.$store.dispatch("form/revert", this.id);
    },
    save(e) {
      if (!e) {
        return false;
      }

      if (e.preventDefault) {
        e.preventDefault();
      }

      if (this.hasChanges === false) {
        return true;
      }

      this.$store
        .dispatch("form/save", this.id)
        .then(() => {
          this.$events.$emit("model.update");
          this.$store.dispatch("notification/success", ":)");
        })
        .catch(response => {
          if (response.code === 403) {
            return;
          }

          if (response.details) {
            this.$store.dispatch("notification/error", {
              message: this.$t("error.form.incomplete"),
              details: response.details
            });
          } else {
            this.$store.dispatch("notification/error", {
              message: this.$t("error.form.notSaved"),
              details: [
                {
                  label: "Exception: " + response.exception,
                  message: response.message
                }
              ]
            });
          }
        });
    }
  }
};
</script>

<style lang="scss">
.k-form-buttons {
  background: $color-notice-on-dark;
}
.k-form-buttons[theme="lock"] {
  background: $color-negative-on-dark;
}
.k-form-buttons .k-view {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.k-form-button {
  font-weight: 500;
  white-space: nowrap;
  line-height: 1;
  height: 2.5rem;
  display: flex;
  padding: 0 1rem;
  align-items: center;
}
.k-form-button:first-child {
  margin-left: -1rem;
}
.k-form-button:last-child {
  margin-right: -1rem;
}
</style>
