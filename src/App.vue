<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <v-app>

    <div v-if="data">
      <!--   Карта   -->
      <v-card class="card">
      <v-data-table
        :headers="headers"
        :items="passedAudits"
        expand
        hide-actions
        disable-initial-sort
        class="elevation-1"
      >
        <template v-slot:items="props">
          <td>{{ props.item.title }}</td>
        </template>
      </v-data-table>
      </v-card>
    </div>

    <v-progress-circular v-if="!data"
        :size="70"
        :width="7"
        color="purple"
        indeterminate
    ></v-progress-circular>

  </v-app>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'app',
    data: function () {
      return {
        data: null,
        audits: [],
        auditsRefs: [],
        // opportunities: this.filterOpportunities,
        // diagnostics: [...this.filterDiagnostics],
        errors: [],
        headers: [
          {
            text: 'Opportunities',
            align: 'left',
            sortable: false,
            value: 'name'
          }
        ]
      };
    },
    methods: {
     getData() {
       axios.get('http://localhost:8080/hardcode_data.json')
         .then(response => {
           this.data = response.data['lighthouseResult'];
           this.audits = Object.keys(response.data['lighthouseResult']['audits']).map(key =>
             response.data['lighthouseResult']['audits'][key]);
           this.auditsRefs = response.data.lighthouseResult.categories.performance.auditRefs;
         })
         .catch(e => {
           this.errors.push(e);
         })
     },
      showPassed(item) {
       let auditItem = this.audits.find(element => element.id === item.id);
        switch (auditItem) {
          case 'manual':
          case 'notApplicable':
            return true;
          case 'error':
          case 'informative':
            return false;
          case 'numeric':
          case 'binary':
          default:
            return Number(auditItem.score) >= 0.9;
        }
      }
     },
    created() {
      this.getData()
    },
    computed: {
      filterOpportunities() {
        return this.auditsRefs.filter(item => item.group === "load-opportunities")
      },
      filterDiagnostics() {
        return this.auditsRefs.filter(item => item.group === "diagnostics")
      },
      passedAudits() {
        return this.auditsRefs.filter(item => (item.group === "load-opportunities" || item.group === "diagnostics")
          && this.showPassed(item))
      }
    }
  };
</script>

<style>
 .card {
   margin: 0 auto;
   width: 500px;
   todo
 }
</style>
