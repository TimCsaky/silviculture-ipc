<template>
  <v-container>
    <BaseSecure>
      <v-alert v-if="error" type="error" tile dense>An error occurred fetching this submission.</v-alert>
      <div v-else>
        <v-container class="review-form">
          <v-stepper>
            <v-row no-gutters>
              <v-col cols="12" xl="8" offset-xl="2">
                <!-- <v-row class="mx-3 mt-5"> -->
                <v-row class="mb-4">
                  <v-col cols="8">
                    <h2>{{ business.name }}</h2>
                    <div>
                      <strong>Submitted:</strong>
                      {{ new Date(ipcPlan.createdAt).toLocaleString() }}
                    </div>
                    <div>
                      <strong>Confirmation ID:</strong>
                      {{ this.ipcPlanId.split('-')[0].toUpperCase() }}
                    </div>
                  </v-col>
                  <v-col cols="4">
                    <GeneratePdfButton :ipcPlanId="this.ipcPlanId" class="pdf-link">
                      <v-btn color="primary">
                        <v-icon class="mr-2">cloud_download</v-icon>PDF
                      </v-btn>
                    </GeneratePdfButton>
                  </v-col>
                </v-row>

                <Step2 :reviewMode="true" />
                <h2 class="review-heading">Before Workers Arrive</h2>
                <Step3 :reviewMode="true" />
                <h2 class="review-heading">After Workers Arrive</h2>
                <Step4 :reviewMode="true" />
                <h2 class="review-heading">If Workers Become Ill</h2>
                <Step5 :reviewMode="true" />
                <h2 class="review-heading">Form Completion Questions</h2>
                <div class="container">
                  <v-form>
                    <v-checkbox
                      :readonly="true"
                      v-model="ipcPlan.certifyAccurateInformation"
                      label="I certify this information to be accurate"
                    ></v-checkbox>
                    <v-checkbox
                      :readonly="true"
                      v-model="ipcPlan.agreeToInspection"
                      label="I agree that my planting camps will be subject to a site inspection"
                    ></v-checkbox>
                  </v-form>
                </div>
              </v-col>
            </v-row>
          </v-stepper>
        </v-container>

        <router-link :to="{ name: 'Admin'}">
          <v-btn color="primary" class="mt-5">Back</v-btn>
        </router-link>
      </div>
    </BaseSecure>
  </v-container>
</template>

<script>
import {mapGetters, mapActions } from 'vuex';

import GeneratePdfButton from '@/components/common/GeneratePdfButton.vue';
import Step2 from '@/components/form/Step2.vue';
import Step3 from '@/components/form/Step3.vue';
import Step4 from '@/components/form/Step4.vue';
import Step5 from '@/components/form/Step5.vue';

export default {
  name: 'Submission',
  props: ['ipcPlanId'],
  components: {
    GeneratePdfButton,
    Step2,
    Step3,
    Step4,
    Step5,
  },
  data() {
    return {
      error: false,
    };
  },
  computed: {
    ...mapGetters('form', ['business', 'contacts', 'covidContact', 'ipcPlan', 'location']),
  },
  methods: {
    ...mapActions('form', ['updateStore']),
  },
  mounted() {
    // update submission data in store
    this.updateStore(this.ipcPlanId);
    // apply vue attributes to improve layout
    document.querySelectorAll('.review-form input, .review-form .v-select, .review-form .ipc-datepicker').forEach(q => {
      q.setAttribute('readonly', 'true');
      q.style.pointerEvents = 'none';
    });
  }
};

</script>
<style lang="scss" scoped>
.review-form {
  padding: 0em;
  font-size: smaller;
  .v-stepper {
    padding: 2em;
  }
  .pdf-link {
    float: right;
    display: block;
  }
  .review-heading {
    margin: 1em 0;
  }
  &::v-deep {
    .container {
      padding: 0px !important;
    }
    h3,
    .v-input--checkbox {
      margin-top: 0.2em !important;
    }
    h4 {
      font-size: 130%;
    }
    .hide-on-review {
      display: none;
    }
  }
}

.review-form:not(.edit-mode) {
  &::v-deep {
    .bus-name-row {
      display: none;
    }
    .ipc-datepicker button {
      display: none !important;
    }
    .v-text-field__details {
      display: none !important;
    }
  }
}
</style>
