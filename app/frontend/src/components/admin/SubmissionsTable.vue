<template>
  <v-card>
    <v-toolbar flat color="grey lighten-3">
      <v-card-title>Form Submissions</v-card-title>
    </v-toolbar>
    <v-container>
      <!-- search input -->
      <div class="ipc-search">
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
          class="pb-5"
        ></v-text-field>
      </div>
      <!-- table alert -->
      <v-alert v-if="showAlert" :type="alertType" tile dense>{{alertMessage}}</v-alert>
      <!-- table header -->
      <v-data-table
        :headers="headers"
        :items="submissions"
        :items-per-page="10"
        :search="search"
        :loading="loading"
        loading-text="Loading... Please wait"
        item-key="confirmationId"
        class="ipc-table"
        @click:row="clickRow"
      >
        <!-- view individual form submission -->
        <template v-slot:item.pdf="{ item }">
          <GeneratePdfButton :ipcPlanId="item.ipcPlanId">
            <v-btn small color="primary">
              <v-icon>cloud_download</v-icon>
            </v-btn>
          </GeneratePdfButton>
        </template>

        <!-- view individual form submission -->
        <template v-slot:item.ipcPlanId="{ item }">
          <router-link :to="{ name: 'Submission', params: { ipcPlanId: item.ipcPlanId } }">
            <v-btn small color="primary">view</v-btn>
          </router-link>
        </template>
      </v-data-table>
    </v-container>
  </v-card>
</template>

<script>
import GeneratePdfButton from '@/components/common/GeneratePdfButton.vue';
import ipcService from '@/services/ipcService';
import Vue from 'vue';

export default {
  name: 'SubmissionsTable',
  components: {
    GeneratePdfButton
  },
  computed: {
    responsiveCell () {
      return (this.$vuetify.breakpoint.name == 'xs') ? 'v-data-table__mobile-table-row' : '';
    }
  },
  data() {
    return {
      // vuetify data table
      search: '',
      headers: [
        { text: 'Submitted', value: 'created' },
        { text: 'Business Name', align: 'start', value: 'name' },
        { text: 'Confirmation ID', align: 'start', value: 'confirmationId' },
        { text: 'PDF', value: 'pdf'},
        { text: '', value: 'ipcPlanId'}
      ],
      submissions: [],
      loading: true,
      showAlert: false,
      alertType: null,
      alertMessage: '',
      expanded: []
    };
  },
  methods: {
    formatDate(date){
      return (date) ? new Date(date).toLocaleString() : 'N/A';
    },
    // get table data from frontend service layer
    getData() {
      ipcService
        .getAllIPCMetaData()
        .then(response => {
          const data = response.data;
          const submissions = Object.keys(data).map(k => {
            let submission = data[k];
            return {
              ipcPlanId: submission.ipcPlan.ipcPlanId,
              pdf: submission.ipcPlan.ipcPlanId,
              name: submission.business.name,
              created: this.formatDate(submission.ipcPlan.createdAt),
              confirmationId: submission.confirmationId,
            };
          });
          if (!submissions.length) {
            this.showTableAlert('info', 'No Submissions found');
          }
          this.submissions = submissions;
        })
        .catch(() => {
          this.showTableAlert('error', 'No response from server');
        });
    },
    generatePdf(ipcPlanId){
      const pdf = `${Vue.prototype.$config.basePath}/${Vue.prototype.$config.apiPath}/ipc/pdf/${ipcPlanId}`;
      window.open(pdf, '_blank');
    },
    clickRow(submission){
      this.$router.push({ name: 'Submission', params: { ipcPlanId: submission.ipcPlanId } });
    },
    showTableAlert(typ, msg) {
      this.showAlert = true;
      this.alertType = typ;
      this.alertMessage = msg;
      this.loading = false;
    },
  },
  mounted() {
    this.getData();
  },
  watch: {
    // hide data table progress bar when submissions have been returned from backend
    submissions() {
      this.loading = false;
    }
  }
};
</script>

<style lang="scss" scoped>
.ipc-search {
  width: 100%;
  max-width: 20em;
  float: right;
}
.ipc-table {
  clear: both;
  cursor: pointer;
}
</style>
