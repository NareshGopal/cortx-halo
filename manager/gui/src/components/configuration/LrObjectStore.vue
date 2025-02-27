<!--
* CORTX-CSM: CORTX Management web.
* Copyright (c) 2022 Seagate Technology LLC and/or its Affiliates
* This program is free software: you can redistribute it and/or modify
* it under the terms of the GNU Affero General Public License as published
* by the Free Software Foundation, either version 3 of the License, or
* (at your option) any later version.
* This program is distributed in the hope that it will be useful,
* but WITHOUT ANY WARRANTY; without even the implied warranty of
* MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
* GNU Affero General Public License for more details.
* You should have received a copy of the GNU Affero General Public License
* along with this program. If not, see <https://www.gnu.org/licenses/>.
* For any questions about this software or licensing,
* please email opensource@seagate.com.
-->
<template>
  <div class="object-store-container config-page">
    <div>
      <div class="page-title">
        Object Store
        <SgtTooltipIcon>
          Administrator can change default limits of the object store and also
          override the SSL certificate packaged with the system.
        </SgtTooltipIcon>
      </div>
      <v-divider></v-divider>
      <div class="page-sub-title">S3 Config</div>
    </div>
    <v-expansion-panels v-model="panel">
      <v-expansion-panel>
        <v-expansion-panel-header><b> Limits </b></v-expansion-panel-header>
        <v-expansion-panel-content class="panel-content">
          <v-form v-model="isLimitsValid">
            <v-row class="field-row">
              <v-col cols="3" class="field-label">
                Max S3 Account
                <SgtTooltipIcon>
                  Maximum limit of S3 accounts that can be created in the
                  system. Exceeding this value will cause an error.
                </SgtTooltipIcon>
              </v-col>
              <v-col cols="4">
                <v-text-field
                  v-model="limit.maxS3Account"
                  :rules="validationRules.number"
                  placeholder="Enter max S3 Account"
                  type="number"
                  min="1"
                  outlined
                  dense
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row class="field-row">
              <v-col cols="3" class="field-label">
                Max IAM user
                <SgtTooltipIcon>
                  Maximum limit of IAM user accounts that can be created in the
                  system. Exceeding this value will cause an error.
                </SgtTooltipIcon>
              </v-col>
              <v-col cols="4">
                <v-text-field
                  v-model="limit.maxIAMUser"
                  :rules="validationRules.number"
                  placeholder="Enter max IAM user"
                  type="number"
                  min="1"
                  outlined
                  dense
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row class="field-row">
              <v-col cols="3" class="field-label">
                Max Bucket
                <SgtTooltipIcon>
                  Maximum limit of S3 buckets that can be created in the system.
                  Exceeding this value will cause an error.
                </SgtTooltipIcon>
              </v-col>
              <v-col cols="4">
                <v-text-field
                  v-model="limit.maxBucket"
                  :rules="validationRules.number"
                  placeholder="Enter max bucket"
                  type="number"
                  min="1"
                  outlined
                  dense
                ></v-text-field>
              </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-row class="field-row">
              <v-col cols="3"></v-col>
              <v-col cols="4" class="button-col">
                <v-btn
                  class="mr-5"
                  color="primary"
                  @click="applyLimit"
                  :disabled="!isLimitsValid"
                  :dark="isLimitsValid"
                  >Apply
                </v-btn>
                <v-btn
                  color="csmdisabled"
                  @click="resetConfirmation()"
                  depressed
                  dark
                  >Reset</v-btn
                >
              </v-col>
            </v-row>
          </v-form>
        </v-expansion-panel-content>
      </v-expansion-panel>
      <v-expansion-panel>
        <v-expansion-panel-header
          ><b> SSL Config </b>
        </v-expansion-panel-header>
        <v-expansion-panel-content class="panel-content">
          <v-row class="field-row">
            <v-col cols="3" class="field-label">SSL certificate status</v-col>
            <v-col cols="4" style="min-height: 75px">
              {{ SSLConfig.status }}
            </v-col>
          </v-row>
          <v-row class="field-row">
            <v-col cols="3" class="field-label">
              SSL certificate upload
              <SgtTooltipIcon>
                The user can upload & install a new certificate which will be
                applied to all the nodes in the cluster.
              </SgtTooltipIcon>
            </v-col>
            <v-col cols="4">
              <SgtDropFile
                :fileName="SSLConfig.SSLCertificateName"
                fileType="*.pem, *.crt, *.ca-bundle, *.cer, *.p7b, *.p7s"
                v-model="SSLFile"
              />
            </v-col>
          </v-row>
          <v-row class="field-row">
            <v-col cols="3"></v-col>
            <v-col cols="4" class="button-col">
              <v-btn
                class="mr-5"
                color="primary"
                @click="installCertificateConfirmation"
                :dark="SSLFile.length > 0"
                :disabled="SSLFile.length < 1"
                >Install Certificate
              </v-btn>
            </v-col>
          </v-row>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import SgtDropdown from "@/lib/components/SgtDropdown/SgtDropdown.vue";
import SgtDropFile from "@/lib/components/SgtDropFile/SgtDropFile.vue";
import { Api } from "@/services/Api";
import SgtDialog from "@/lib/components/SgtDialog/SgtDialog.vue";
import { SgtDialogModel } from "@/lib/components/SgtDialog/SgtDialog.model";
import { create } from "vue-modal-dialogs";
import SgtTooltipIcon from "@/lib/components/SgtTooltipIcon/SgtTooltipIcon.vue";

@Component({
  name: "LrObjectStore",
  components: { SgtDropdown, SgtDropFile, SgtTooltipIcon },
})
export default class LrObjectStore extends Vue {
  panel = 0;
  limit = {};
  resetModal = create<SgtDialogModel>(SgtDialog);
  SSLConfig = {
    status: null,
    SSLCertificateName: null,
  };
  SSLFile = [];
  SSLModal = create<SgtDialogModel>(SgtDialog);

  isLimitsValid = false;

  validationRules = {
    number: [
      (val: number) => !!val || "This field is required",
      (val: string) => /^\d+$/.test(val) || "Invalid value",
    ],
  };

  mounted() {
    this.getLimits();
    this.getSSLConfig();
  }

  async getLimits() {
    const resp: any = await Api.getData("config/limit", { isDummy: true });
    this.limit = JSON.parse(JSON.stringify(resp.data));
  }

  async resetLimitValues() {
    //API call to reset the limit value followed by the getLimits call
    this.getLimits();
  }

  async resetConfirmation() {
    const result = await this.resetModal({
      modalTitle: "Confirmation",
      modalContent: `Are you sure you want to reset the data?`,
      modalType: "prompt",
      modalContentType: "html",
    }).then(async (resp) => {
      if (resp === "yes") {
        this.resetLimitValues();
      }
    });
  }

  async getSSLConfig() {
    const resp: any = await Api.getData("config/SSLConfig", { isDummy: true });
    this.SSLConfig = resp.data;
  }

  applyLimit() {
    //api call
  }

  async installCertificateConfirmation() {
    const result = await this.SSLModal({
      modalTitle: "Confirmation",
      modalContent: `Click Yes to install the SSL certificate. After installing the SSL certificate, you must login again.`,
      modalType: "prompt",
      modalContentType: "html",
    }).then((resp) => {
      this.installCertificate();
      console.log(resp);
    });
  }

  async installCertificate() {
    //API call to install the certificate

    //Use below popup when the API call succeeds
    await this.SSLModal({
      modalTitle: "Success",
      modalContent: `SSL certificate has been successfully installed.`,
      infoType: "neutral",
      modalType: "message",
      modalContentType: "text",
      okButtonLabel: "Ok",
    });

    //Use below popup when the API call fails
    await this.SSLModal({
      modalTitle: "Error",
      modalContent: `Error in installing certificate`, //Display the error message from API response
      infoType: "neutral",
      modalType: "message",
      modalContentType: "text",
      okButtonLabel: "Ok",
    });
  }
}
</script>
<style lang="scss" scoped></style>
