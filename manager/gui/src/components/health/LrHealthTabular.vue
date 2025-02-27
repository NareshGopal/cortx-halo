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
  <div class="health-tabular-container">
    <SgtDataTable
      ref="healthDetailsTable"
      :headers="healthTableConfig.healthTable.headers"
      :records="clusterHealthData"
      :searchConfig="healthTableConfig.searchConfig"
      @onRowHover="handleHover"
    >
      <template v-slot:status="{ data }">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-avatar
              v-bind="attrs"
              v-on="on"
              class="status-icon"
              :color="getColor(data.status)"
              size="16"
            ></v-avatar>
          </template>
          <span class="text-capitalize">{{ data.status }}</span>
        </v-tooltip>
      </template>

      <template v-slot:actionColumn="{ data }">
        <div class="action-icons-wrapper">
          <div class="all-icons-container">
            <div
              class="action-icons-container"
              v-if="data.resource === 'node' && data.rowIdx === rowHoverIndex"
            >
              <template v-for="(icon, index) in getIconList(data)">
                <SgtSvgIcon
                  :key="index"
                  :icon="icon.path"
                  :hoverIcon="icon.hoverPath"
                  :disabled="icon.disabled"
                  :tooltip="icon.tooltip"
                  @click="actionIconHandler(icon.action, data)"
                />
              </template>
            </div>
          </div>
        </div>
      </template>
    </SgtDataTable>

    <v-dialog v-model="displayInfoModal" width="500">
      <v-card class="pt-3">
        <v-card-text class="mt-2">
          Resource: {{ resourceInfo.resource.toUpperCase() }}
          <br />
          ID: {{ resourceInfo.id }}
          <br />
          Status: {{ resourceInfo.status.toUpperCase() }}
          <br />
          Time: {{ formattedTime(resourceInfo.last_updated_time * 1000) }}
          <br />
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            class="white--text"
            color="#6ebe49"
            @click="displayInfoModal = false"
            >OK</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script lang="ts">
import { Component, Mixins } from "vue-property-decorator";
import ClusterManagementMixin from "../../mixins/ClusterManagement";
import { Api } from "../../services/Api";
import { lrHealthConst } from "./LrHealthTabular.constant";
import { IResource } from "./LrHealthTabular.model";
import SgtDataTable from "@/lib/components/SgtDataTable/SgtDataTable.vue";
import SgtSvgIcon from "@/lib/components/SgtSvgIcon/SgtSvgIcon.vue";
import { formatTime } from "../../utils/CommonUtilFunctions";

@Component({
  name: "LrHealthTabular",
  components: { SgtDataTable, SgtSvgIcon },
})
export default class LrHealthTabular extends Mixins(ClusterManagementMixin) {
  public hideFilter: boolean = true;
  public healthQueryParams: any = {};
  public healthTableHeaderList: any[] = [];
  public clusterHealthData: any = [];
  public displayInfoModal: boolean = false;
  public healthTableConfig: any = lrHealthConst;
  public rowHoverIndex: number = -1;
  public resourceInfo: IResource = {
    id: "",
    last_updated_time: "",
    resource: "",
    status: "",
  };
  public formattedTime(date: string) {
    return formatTime(date);
  }
  public mounted() {
    this.getHealthData();
  }
  public handleHover(index: number) {
    this.rowHoverIndex = index;
  }
  public async getHealthData() {
    this.healthQueryParams = {
      response_format: "flattened",
      offset: 1,
      limit: 0,
    };
    // this.$store.dispatch("systemConfig/showLoader", "Fetching health...");
    const res: any = await Api.getData("/health/health-tabular", {
      isDummy: true,
    });
    this.clusterHealthData = res.data;
    // this.$store.dispatch("systemConfig/hideLoader");
  }

  public getIconList(data: IResource) {
    return [
      {
        path: `start-${data.status}.svg`,
        hoverPath: `start-${data.status}.svg`,
        disabled: data.status === "online",
        action: "start",
        tooltip: "Start Node",
      },
      {
        path: `stop-${data.status}.svg`,
        hoverPath: `stop-${data.status}.svg`,
        disabled: data.status === "offline" || data.status === "failed",
        action: "stop",
        tooltip: "Stop Node",
      },
      {
        path: `poweroff-${data.status}.svg`,
        hoverPath: `poweroff-${data.status}.svg`,
        action: "poweroff",
        tooltip: "Server Power Off",
      },
      {
        path: `powerandstorageoff-${data.status}.svg`,
        hoverPath: `powerandstorageoff-${data.status}.svg`,
        action: "powerandstorageoff",
        tooltip: "Server and Storage Off",
      },
    ];
  }

  public actionIconHandler(action: string, data: any) {
    //Perform API calls for respective actions
    console.log(action);
    console.log(data);
  }

  get actionsCallback() {
    return {
      startNodeAction: (resource: any) => this.performAction(resource, "start"),
      stopNodeAction: (resource: any) => this.performAction(resource, "stop"),
      powerOffAction: (resource: any) =>
        this.performAction(resource, "poweroff"),
      powerAndStorageOffAction: (resource: any) =>
        this.performAction(resource, "powerandstorageoff"),
    };
  }

  getColor(item: "online" | "offline" | "degraded" | "failed" | "unknown") {
    return this.healthTableConfig.severityList[item];
  }

  startNode(resource: IResource) {
    //API call to start the node
  }

  stopNode(resource: IResource) {
    //API call to stop the node
  }

  serverPowerOff(resource: IResource) {
    //API call to turn the server off
  }

  serverAndStorageOff(resource: IResource) {
    //API call to turn the server and storage off
  }
}
</script>
<style lang="scss" scoped>
.health-tabular-container {
  .status-icon {
    margin-left: 1em;
  }
  .action-icons-wrapper {
    display: flex;
    justify-content: flex-end;
    .all-icons-container {
      position: relative;

      .action-icons-container {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        gap: 10px;
        position: absolute;
        right: 0;
        top: -12px;
      }
    }
  }
}
.health-tabular-container::v-deep .sgt-data-table {
  .v-data-table {
    .footer-wrapper {
      margin-bottom: 5px !important;
    }
  }
}
</style>
