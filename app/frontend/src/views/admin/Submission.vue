<template>
  <v-container>
    <BaseSecure>
      <v-alert v-if="error" type="error" tile dense>An error occurred fetching this submission.</v-alert>
      <div v-else>
        <router-link :to="{ name: 'Admin'}">
          <v-btn color="primary" class="mb-5">Back</v-btn>
        </router-link>

        <GeneratePdfButton :ipcPlanId="this.ipcPlanId" class="pdf-link">
          <v-icon color="primary" large>picture_as_pdf</v-icon>
        </GeneratePdfButton>

        <v-container class="review-form">
          <v-stepper>
            <v-row no-gutters>
              <v-col cols="12" xl="8" offset-xl="2">
                <div class="container">
                  <h2>{{ business.name }}</h2>
                  <div>
                    <strong>Submitted:</strong>
                    {{ new Date(ipcPlan.createdAt).toLocaleString() }}
                  </div>
                  <div>
                    <strong>Confirmation ID:</strong>
                    {{ ipcPlan.ipcPlanId.split('-')[0].toUpperCase() }}
                  </div>
                </div>
                <Step2 :reviewMode="true" />
                <h2 class="review-heading">Before Workers Arrive</h2>
                <Step3 :reviewMode="true" />
                <h2 class="review-heading">After Workers Arrive</h2>
                <Step4 :reviewMode="true" />
                <h2 class="review-heading">If Workers Become Ill</h2>
                <Step5 :reviewMode="true" />
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
    document.querySelectorAll('.review-form input').forEach(q => {
      q.setAttribute('readonly', 'true');
    });
  }
};

</script>
<style lang="scss" scoped>
.review-form {
  font-size: smaller;
  .container:nth-child(2) {
    padding: 0px !important;
  }
  .review-heading {
    margin: 12px;
  }
  background-color: #efefef;
  &::v-deep {
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
    .bus-name-row {
      display: none;
    }
  }
}
.pdf-link {
  float: right;
  display: block;
}
</style>
