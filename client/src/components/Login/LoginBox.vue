<template>
  <v-layout
    class="elevation-10 cluster"
    :class="{ clickable: !cluster.error }"
    :data-cluster="clusterName"
    column
    :style="cssProps"
    @click="login"
  >
    <v-flex>
      <div class="cluster-name">
        <status-dot :status="cluster.status" :size="20" />
        {{ clusterName }}
      </div>
    </v-flex>
    <v-flex>
      <v-layout class="info-box" justify-end align-end>
        <v-flex v-if="!cluster.error">
          <span class="key">Version:</span>
          {{ cluster.versions.join(" , ") }}<br />
          <span class="key">Nodes:</span>
          {{ cluster.numNodes }}<br />
          <span class="key">Docs:</span>
          {{ cluster.docCount.toLocaleString() }}<br />
          <span class="key">Size:</span>
          {{
            (cluster.storeSizeInBytes / 1024 / 1024 / 1024).toFixed(1)
          }}gb<br />
          <span class="key">JVM Name:</span>
          {{ cluster.jvmName }}<br />
        </v-flex>
        <v-flex v-else class="error-text">{{ cluster.error.args.body }}</v-flex>
        <v-flex shrink>
          <v-btn icon text :disabled="!!cluster.error">
            <v-icon color="white">arrow_forward</v-icon>
          </v-btn>
        </v-flex>
      </v-layout>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";
import { GET } from "../../requests";
import StatusDot from "../StatusDot.vue";
import { clusterStatus } from "../../enums";
import colors from "vuetify/es5/util/colors";

export default {
  props: {
    clusterName: {
      type: String,
      required: false
    }
  },
  components: { StatusDot },
  data() {
    return {
      cluster: { versions: [], docCount: 0, status: clusterStatus.loading }
    };
  },
  methods: {
    ...mapMutations(["selectCluster"]),
    login() {
      if (this.cluster.error || this.cluster.status === clusterStatus.loading) {
        return;
      }
      this.selectCluster({
        cluster: this.clusterName,
        version: this.cluster.versions[0]
      });
    }
  },
  async created() {
    try {
      this.cluster = await GET(
        `/api/v1/cluster/info/${this.clusterName}`,
        false
      );
    } catch (error) {
      this.cluster.status = clusterStatus.error;
      this.cluster.error = error;
    }
  },
  computed: {
    cssProps() {
      const { dark } = this.$vuetify.theme;
      const { blueGrey } = colors;
      return {
        "--header-color": dark ? blueGrey.darken1 : blueGrey.lighten1,
        "--body-color": dark ? blueGrey.darken2 : blueGrey.lighten2
      };
    }
  }
};
</script>

<style scoped>
.clickable {
  cursor: pointer;
}
.cluster-name {
  background-color: var(--header-color);
  text-align: center;
  font-size: 30px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  padding: 5px 0;
}
.key {
  font-weight: bold;
}
.cluster {
  margin: 20px;
  border-radius: 10px;
  background-color: var(--body-color);
  height: 180px;
  color: white;
}

.info-box {
  padding-left: 20px;
}

.error-text {
  overflow: hidden;
  height: 100px;
}
.error-text:hover {
  overflow: unset;
}
</style>
