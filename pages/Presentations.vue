<template>
  <VRow justify="center" style="max-width: 1100px; margin: auto;">
    <VCol cols="12">
      <h1 class="text-center">Presentations</h1>
    </VCol>
    <VCol cols="12">
      <h3>Filters:</h3>
      <PresentationFilters :value="filters" />
    </VCol>
    <VCol cols="12">
      <h3>Presentations by themes:</h3>
      <VExpansionPanels v-if="filteredPresentationsCount > 0">
        <div
          v-for="(yearPresentation, theme) in presentationFiltered"
          :key="theme"
          style="width: 100%"
        >
          <div v-if="filteredPresentationsCountByTheme(theme) > 0">
            <!-- Title actually show it like this: Theme (3 presentations) -->
            <h2 class="text-decoration-underline mb-3 text-center">
              {{ theme }} ({{ filteredPresentationsCountByTheme(theme) }}
              {{
                pluralize(
                  'presentation',
                  filteredPresentationsCountByTheme(theme)
                )
              }}):
            </h2>
            <div
              v-for="(presentations, year) in yearPresentation"
              :key="`${theme}-${year}`"
            >
              <!--<h3>{{ year }}</h3>-->
              <PresentationItem
                v-for="presentation in presentations"
                :key="presentation.title"
                :presentation="presentation"
                :year="year"
              />
            </div>
          </div>
        </div>
      </VExpansionPanels>
      <div v-else>
        <p>No presentation match with the current filters you selected.</p>
      </div>
    </VCol>
  </VRow>
</template>

<script>
import presentationData from '@/assets/json/presentations'
import PresentationItem from '@/components/PresentationItem'
import PresentationFilters from '@/components/PresentationFilters'

export default {
  name: 'PresentationList',
  components: {
    PresentationItem,
    PresentationFilters
  },
  data() {
    return {
      filters: {
        theme: '',
        year: '',
        author: '',
        event: '',
        country: '',
        language: '',
        title: ''
      }
    }
  },
  computed: {
    presentationFiltered() {
      // The filter loop over themes then years then presentations to filter them. With this logic we iterate only once on all presentations. The logic look like this:
      // Is theme in the theme filtered ?
      //    yes -> Is the year in the year filtered ?
      //        yes -> is the presentation has the same value that the filter ?
      //            yes -> display this presentation
      //            no -> remove this presentation
      //        no -> remove all the presentation of this year
      //    no -> remove all presentation of this theme
      return this.filterByTheme(presentationData, this.filters)
    },
    filteredPresentationsCount() {
      // This computed count all the presentation filtered
      return Object.keys(this.presentationFiltered).reduce((count, theme) => {
        return count + this.filteredPresentationsCountByTheme(theme)
      }, 0)
    }
  },
  watch: {
    '$route.query.title'(title) {
      if (!title) {
        return
      }
      this.filters.title = title
    }
  },
  created() {
    if (this.$route.query.title) {
      this.filters.title = this.$route.query.title
    }
  },
  methods: {
    filterByTheme(presentationByTheme, filters) {
      // presentationByTheme is presentationData object
      let allowed
      // If we want to filter by theme we assign it. If not we assign all the key to not filter it
      if (filters.theme) {
        allowed = [filters.theme]
      } else {
        allowed = Object.keys(presentationByTheme)
      }
      // This code basically just filter by key
      return Object.keys(presentationByTheme)
        .filter((key) => allowed.includes(key))
        .reduce((obj, key) => {
          obj[key] = this.filterByYear(presentationByTheme[key], filters)
          return obj
        }, {})
    },
    filterByYear(presentationByYear, filters) {
      // presentationByYear look like {2019: [...], 2020: [...]}
      // If we want to filter by year we assign it. If not we assign all the key to not filter it
      let allowed
      if (filters.year) {
        allowed = [filters.year]
      } else {
        allowed = Object.keys(presentationByYear)
      }
      // we filter presentation by year and when we set the presentations data we filter them directly to avoid multiple loop
      return Object.keys(presentationByYear)
        .filter((key) => allowed.includes(key))
        .reduce((obj, key) => {
          obj[key] = this.filterPresentations(presentationByYear[key], filters)
          return obj
        }, {})
    },
    filterPresentations(presentations, filters) {
      // prepare filter by multiple value
      return presentations.filter((presentation) => {
        return this.presentationMatchFilter(presentation, filters)
      })
    },
    presentationMatchFilter(presentation, filters) {
      if (filters.author && filters.author !== presentation.author) {
        return false
      }
      if (filters.country && filters.country !== presentation.country) {
        return false
      }
      if (filters.language && filters.language !== presentation.language) {
        return false
      }
      if (filters.event && filters.event !== presentation.event.name) {
        return false
      }
      if (
        filters.title &&
        !presentation.title.toLowerCase().includes(filters.title.toLowerCase())
      ) {
        return false
      }
      return true
    },
    filteredPresentationsCountByTheme(theme) {
      // This methode count all the presentation filtered by theme
      return Object.keys(this.presentationFiltered[theme]).reduce(
        (count, year) => {
          return count + this.presentationFiltered[theme][year].length
        },
        0
      )
    },
    pluralize(text, number) {
      return number > 1 ? text + 's' : text
    }
  }
}
</script>

<style></style>
