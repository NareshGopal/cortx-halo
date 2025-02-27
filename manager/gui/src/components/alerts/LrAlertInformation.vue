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
  <div class="alert-details-container" v-if="alert">
    <div v-if="alert && alertExtendedInfo">
      <SgtLabel name="Id" :value="alertExtendedInfo.resource_id" />
      <SgtLabel name="Name" :value="alert.module_type" />
      <SgtLabel name="Cluster" :value="alertExtendedInfo.cluster_id" />
      <SgtLabel name="Site" :value="alertExtendedInfo.site_id" />
      <SgtLabel name="Rack" :value="alertExtendedInfo.rack_id" />
      <SgtLabel name="Host Name" :value="alert.hostname" />
    </div>
    <div>
      <SgtLabel name="Resource Type" :value="alertExtendedInfo.resource_type" />
      <SgtLabel name="State" :value="alert.state" />
      <SgtLabel name="Created Time" :value="new Date(alert.created_time*1000)" />
      <SgtLabel name="Updated Time" :value="new Date(alert.updated_time*1000)" />
    </div>

    <div>
      <template v-if="alert.module_type === 'logical_volume'">
        <SgtLabel name="Volume Group" :value="alert.volume_group" />
        <SgtLabel name="Volume Name" :value="alert.name" />
      </template>
      <template v-else-if="alert.module_type === 'system'">
        <SgtLabel name="Version" :value="alert.version" />
        <SgtLabel name="Node Name" :value="alert.name" />
      </template>
      <template v-else-if="alert.module_type === 'volume'">
        <SgtLabel name="Size" :value="alert.volume_size" />
        <SgtLabel name="Total Size" :value="alert.volume_total_size" />
      </template>
      <template v-else-if="alert.module_type === 'current'">
        <SgtLabel name="Sensor Name" :value="alert.name" />
      </template>
      <template v-else-if="alert.module_type === 'controller'">
        <SgtLabel name="Serial Number" :value="alert.serial_number" />
      </template>
    </div>

    <div>
      <template v-if="alert.module_name === 'enclosure:fru:psu'">
        <SgtLabel name="Location" :value="alert.location" />
      </template>
      <template
        v-if="alert.module_name === 'enclosure:fru:fan' || alert.module_name === 'enclosure:fru:sideplane'"
      >
        <SgtLabel name="Name" :value="alert.Name" />
        <SgtLabel name="Location" :value="alert.location" />
      </template>
      <template v-if="alert.module_name === 'enclosure:fru:disk'">
        <SgtLabel name="Serial Number" :value="alert.serial_number" />
        <SgtLabel name="Size" :value="alert.volume_size" />
      </template>
    </div>

    <template v-if="alertEventDetails.length > 0">
      <div v-for="(event_detail, i) in alertEventDetails" v-bind:key="'event_detail_' + i">
        <SgtLabel name="Name" :value="event_detail.name" />
        <SgtLabel name="Reason" :value="event_detail.event_reason" />
        <SgtLabel
          v-if="event_detail.event_recommendation.length > 0"
          name
          :value="alerts.recommendations"
        />
      </div>
    </template>

    <div style="text-align:right; padding:1rem 0">
      <div class="alert-info">
        <img v-if="alert.resolved" :src="require('@/assets/icons/resolved-disabled.svg')" />
        <img v-else :src="require('@/assets/icons/resolved-disabled.svg')" />
        <span>Resolved</span>
      </div>
      <div class="alert-info">
        <img v-if="alert.acknowledged" :src="require('@/assets/icons/alerts-disabled.svg')" />
        <img v-else :src="require('@/assets/icons/alert-green.svg')" />
        <span>Acknowledged</span>
      </div>
      <div class="alert-info">
        <SgtSvgIcon
          icon="more-info.svg"
          @click="showAlertDetailsDialog = true"
          class="action-btn-block"
        />
        <SgtSvgIcon
          icon="comment-default.svg"
          hoverIcon="comment-hover.svg"
          @click="showAlertCommentsDialog = true"
          class="action-btn-block"
        />
        <SgtSvgIcon
          icon="alert-green.svg"
          hoverIcon="alert-hover.svg"
          @click="acknowledge"
          class="action-btn-block"
        />
      </div>
    </div>

    <div>
      <LrAlertDialog
        v-if="alertDetails && alertExtendedInfo && Object.keys(alertExtendedInfo).length>0"
        :modalTitle="'Alert Details'"
        :modalData="alertExtendedInfo"
        :showAlertDetailsDialog.sync="showAlertDetailsDialog"
      />
    </div>
    <div>
      <LrAlertComments
        v-if="alertDetails"
        :id="alertDetails.alert_uuid"
        :showAlertCommentsDialog.sync="showAlertCommentsDialog"
      />
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Vue, Prop, Mixins, Watch } from "vue-property-decorator";
import SgtLabel from "@/lib/components/SgtLabel/SgtLabel.vue";
import LrAlertDialog from "./LrAlertDialog.vue";
import LrAlertComments from "./LrAlertComments.vue";
import SgtSvgIcon from "@/lib/components/SgtSvgIcon/SgtSvgIcon.vue";

@Component({
	name: "LrAlertInformation",
	components: { SgtLabel, LrAlertDialog, SgtSvgIcon, LrAlertComments },
})
export default class LrAlertInformation extends Vue {
	@Prop({ required: true }) private alert: any;
	public alertEventDetails: any = [];
	public alertExtendedInfo: any = {};
	public alertDetails: any = {};
	public showAlertDetailsDialog = false;
	public showAlertCommentsDialog = false;
	public addCommentForm = {
		comment_text: "",
	};

	public async mounted() {
		try {
			if (this.alert.extended_info) {
				this.alertDetails = JSON.parse(this.alert.extended_info);
				this.alertExtendedInfo = this.alertDetails.info;
			}
			let tempAlertEventDetails = [];
			if (this.alert.event_details) {
				tempAlertEventDetails = JSON.parse(this.alert.event_details);
			}
			if (tempAlertEventDetails.length > 0) {
				tempAlertEventDetails.forEach((event_detail: any) => {
					const alertEventDetail = {
						name: event_detail.name,
						event_reason: event_detail.event_reason,
						event_recommendation: event_detail.event_recommendation.split("-"),
						showRecommendation: false,
					};
					this.alertEventDetails.push(alertEventDetail);
				});
			} else {
				this.alertEventDetails.push({
					name: this.alertExtendedInfo.resource_id
						? this.alertExtendedInfo.resource_id
						: "",
					event_reason: this.alert.description,
					event_recommendation: this.alert.health_recommendation
						? this.alert.health_recommendation.split("-")
						: [],
					showRecommendation: false,
				});
			}
		} catch (e) {
			// tslint:disable-next-line: no-console
			console.log(e);
		}
	}

	comment() {
		//comment action
		this.showAlertCommentsDialog = true;
	}
	acknowledge() {
		//acknowledge action
	}
}
</script>

<style lang="scss" scoped >
.alert-details-container {
	margin: 0;
}
.alert-info {
	display: inline-block;
	padding-left: 1rem;
	span {
		vertical-align: super;
	}
	.action-btn-block {
		padding-left: 1rem;
	}
}
</style>