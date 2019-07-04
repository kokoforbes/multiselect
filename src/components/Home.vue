<template>
  <div class="home">
     <multiselect :options="options" :value="optionsProxy" @input="updateSelected" @search-change="searchQuery" :show-labels="true" :multiple="true" :searchable="true" :close-on-select="true" placeholder="Search" :custom-label="customLabel" track-by="name"
    :loading="showLoadingSpinner">
    </multiselect>

    <ul class="resources-list">
      <template v-for="(resource, index) in selectedResources">
        <li class="resource-item" :key="resource.id" :data-index="index">
          <div class="resource-info">
            <div class="resource-title" :id="index">
              <span>{{ resource.name }}</span>
              <span class="version">{{ resource.version }}</span>
            </div>
            <div class="resource-description">
              <span>{{ resource.description }}</span>
            </div>
            <div class="resource-url">
              <a :href="resource.latest" target="_blank">{{ resource.latest }}</a>
            </div>
          </div>
          <div class="delete-controls" v-on:click.prevent="removeDependency(index)">
            <i class="fas fa-times fa-fw"></i>
          </div>
        </li>
      </template>
    </ul>
  </div>
</template>

<script>
import Multiselect from 'vue-multiselect'

export default {
  name: 'Home',
  components: {
    Multiselect
  },
  data() {
    return {
      options: [],
      optionsProxy: [],
      selectedResources: [],
      showLoadingSpinner: false
    }
  },

  methods: {
  customLabel(option) {
    return `${option.name} — ${option.version}`
  },

  updateSelected(value) {
    value.forEach((resource) => {
      // Adds selected resources to array
      this.selectedResources.push(resource)
    })
    // Clears selected array
    // This prevents the tags from being displayed
    this.optionsProxy = []
  },

  cdnRequest(value) {
    this.$http.get(`https://api.cdnjs.com/libraries?search=${value}&fields=version,description`)
      .then( response => {
        // get body data
        this.options = []
        response.body.results.forEach((object) => {
          this.options.push(object)
        })
        this.showLoadingSpinner = false
      }, response => {
        // error callback
      })
  },

  searchQuery(value) {
    this.showLoadingSpinner = true
    // GET
    this.cdnRequest(value);
  },
  
  removeDependency(index) {
    this.selectedResources.splice(index, 1)
  }
}
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.home {
    width: 50vw;
    margin: 0 auto;
}
 @media only screen and (max-width: 800px) {
    .home{
      width: 90vw;
      margin: 0 auto;
    }
  }
.resources-list {
  margin-top: 15px;
  padding: 0;
  list-style: none;
}
.resources-list li {
  display: flex;
  align-items: center;
  min-height: 50px;
  padding: 10px 40px 10px 0;
  border-bottom: 1px solid rgba(51, 51, 51, 0.1);
  position: relative;
}
.resources-list li:last-child {
  border-bottom: none;
}
.resources-list li .resource-title {
  font-size: 1em;
  color: #333;
}
.resources-list li .version {
  opacity: 0.7;
  margin-left: 5px;
  font-size: 75%;
}
.resources-list li .resource-description,
.resources-list li .resource-url {
  font-size: 0.8em;
  color: #999;
  margin-top: 5px;
}
.resources-list li .resource-url {
  margin-top: 0;
}
.resources-list li .resource-description:empty {
  display: none;
}
.resources-list li .delete-controls {
  position: absolute;
  right: 0;
  width: 40px;
  text-align: center;
  color: #999;
  cursor: pointer;
}
.resources-list li .delete-controls:hover,
.resources-list li .delete-controls:focus {
  color: red;
}
</style>
