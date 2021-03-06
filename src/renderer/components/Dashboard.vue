<template>
  <grid>
    <aside class="sidebar__menu" slot="sidebar">
      <ul>
        <li v-for="project in projects" :key="project.id">
          <router-link :to="{ name: 'dashboard', params: { id: project.id } }">
            <span :class="`status status--${$store.getters.projectStatus(project.id)}`"></span>
            {{ $store.getters.projectDirName(project.id) }}
          </router-link>
        </li>
      </ul>
    </aside>

    <div class="sidebar__actions" slot="sidebar">
      <router-link to="/settings" class="icon is-medium" title="Settings">
        <span v-show="updateAvailable" class="badge"></span>
        <i class="fa fa-lg fa-cog"></i>
      </router-link>
    </div>

    <project :project="project"></project>
  </grid>
</template>

<script>
import { mapGetters, mapActions, mapState } from "vuex";
import Grid from "./Grid";
import Project from "./Dashboard/Project";
import Container from "../utils/docker-container";
import Mousetrap from "mousetrap";

export default {
  name: "dashboard",
  components: { Grid, Project },
  methods: {
    previousProject() {
      const idx = this.activeProject;
      this.$router.push({
        name: "dashboard",
        params: {
          id: idx === 0 ? this.projects.length - 1 : idx - 1
        }
      });
    },
    nextProject() {
      const idx = this.activeProject;
      this.$router.push({
        name: "dashboard",
        params: {
          id: idx === this.projects.length - 1 ? 0 : idx + 1
        }
      });
    },
    ...mapActions(["fetchContainers"])
  },
  created() {
    this.fetchContainers();
  },
  mounted() {
    Mousetrap.bind("meta+shift+[", this.previousProject);
    Mousetrap.bind("ctrl+shift+tab", this.previousProject);
    Mousetrap.bind("meta+shift+]", this.nextProject);
    Mousetrap.bind("ctrl+tab", this.nextProject);
  },
  computed: {
    activeProject() {
      return parseInt(this.$route.params.id, 10);
    },

    // Get the active project object
    project() {
      return this.projects[this.activeProject];
    },

    ...mapState({
      updateAvailable: state => state.App.updateAvailable,
      projects: state => state.Project.projects
    })
  }
};
</script>

<style lang="scss" scoped>
.status {
  --size: 0.5em;
  background-color: transparent;
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: var(--size);
  display: inline-block;
  height: var(--size);
  margin-right: 0.2em;
  width: var(--size);

  &--running {
    background-color: #ddeeed;
  }

  &--partial {
    background-color: #ffdd57;
  }
}

.sidebar__actions a {
  position: relative;
}

.badge {
  --size: 0.5em;
  background-color: #ff3860;
  border-radius: var(--size);
  display: inline-block;
  position: absolute;
  right: 0.3em;
  top: 0.3em;
  height: var(--size);
  width: var(--size);
}
</style>
