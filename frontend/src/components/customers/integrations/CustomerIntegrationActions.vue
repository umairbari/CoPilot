<template>
	<div class="contents">
		<n-button v-if="isDeployEnabled" :loading="loading" type="success" :size secondary @click.stop="provision()">
			<template #icon>
				<Icon :name="DeployIcon"></Icon>
			</template>
			Deploy
		</n-button>

		<n-button v-if="!hideDeleteButton" :size type="error" ghost :loading="loadingDelete" @click.stop="handleDelete">
			<template #icon>
				<Icon :name="DeleteIcon" :size="15"></Icon>
			</template>
			Delete
		</n-button>
	</div>
</template>

<script setup lang="ts">
import type { Size } from "naive-ui/es/button/src/interface"
import type { ApiCommonResponse } from "@/types/common.d"
import type { CustomerIntegration } from "@/types/integrations.d"
import { NButton, useDialog, useMessage } from "naive-ui"
import { computed, ref, watch } from "vue"
import Api from "@/api"
import Icon from "@/components/common/Icon.vue"
import { handleDeleteIntegration } from "./utils"

const { integration, hideDeleteButton, size } = defineProps<{
	integration: CustomerIntegration
	hideDeleteButton?: boolean
	size?: Size
}>()

const emit = defineEmits<{
	(e: "startLoading"): void
	(e: "stopLoading"): void
	(e: "deployed"): void
	(e: "deleted"): void
}>()

const DeployIcon = "carbon:deploy"
const DeleteIcon = "ph:trash"

const dialog = useDialog()
const message = useMessage()

const loadingProvision = ref(false)
const loadingDelete = ref(false)
const loading = computed(() => loadingProvision.value || loadingDelete.value)
const serviceName = computed(() => integration.integration_service_name)
const customerCode = computed(() => integration.customer_code)
const isOffice365 = computed(() => serviceName.value === "Office365")
const isMimecast = computed(() => serviceName.value === "Mimecast")
const isCrowdstrike = computed(() => serviceName.value === "Crowdstrike")
const isDuo = computed(() => serviceName.value === "DUO")
const isDarktrace = computed(() => serviceName.value === "Darktrace")
const isBitdefender = computed(() => serviceName.value === "BitDefender")
const isCato = computed(() => serviceName.value === "CATO")
const isDefenderForEndpoint = computed(() => serviceName.value === "DefenderForEndpoint")
const isDeployEnabled = computed(
	() =>
		(isOffice365.value ||
			isMimecast.value ||
			isCrowdstrike.value ||
			isDuo.value ||
			isDarktrace.value ||
			isBitdefender.value ||
			isCato.value ||
      isDefenderForEndpoint.value) &&
		!integration.deployed
)

watch(loading, val => {
	if (val) {
		emit("startLoading")
	} else {
		emit("stopLoading")
	}
})

function provision() {
	let apiCall: Promise<ApiCommonResponse> | null = null

	if (isOffice365.value) {
		apiCall = Api.integrations.office365Provision(customerCode.value, serviceName.value)
	}
	if (isMimecast.value) {
		apiCall = Api.integrations.mimecastProvision(customerCode.value, serviceName.value)
	}
	if (isCrowdstrike.value) {
		apiCall = Api.integrations.crowdstrikeProvision(customerCode.value, serviceName.value)
	}
	if (isDuo.value) {
		apiCall = Api.integrations.duoProvision(customerCode.value, serviceName.value)
	}
	if (isDarktrace.value) {
		apiCall = Api.integrations.darktraceProvision(customerCode.value, serviceName.value)
	}
	if (isBitdefender.value) {
		apiCall = Api.integrations.bitdefenderProvision(customerCode.value, serviceName.value)
	}
	if (isCato.value) {
		apiCall = Api.integrations.catoProvision(customerCode.value, serviceName.value)
	}
  if (isDefenderForEndpoint.value) {
    apiCall = Api.integrations.defenderForEndpointProvision(customerCode.value, serviceName.value)
  }

	if (!apiCall) {
		return
	}

	loadingProvision.value = true

	apiCall
		.then(res => {
			if (res.data.success) {
				emit("deployed")
				message.success(res.data?.message || "Customer integration successfully deployed.")
			} else {
				message.warning(res.data?.message || "An error occurred. Please try again later.")
			}
		})
		.catch(err => {
			message.error(err.response?.data?.message || "An error occurred. Please try again later.")
		})
		.finally(() => {
			loadingProvision.value = false
		})
}

function handleDelete() {
	handleDeleteIntegration({
		integration,
		cbBefore: () => {
			loadingDelete.value = true
		},
		cbSuccess: () => {
			emit("deleted")
		},
		cbAfter: () => {
			loadingDelete.value = false
		},
		message,
		dialog
	})
}
</script>
