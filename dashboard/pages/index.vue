<script setup lang="ts">

definePageMeta({ layout: 'dashboard' });


const route = useRoute();
const { project, projectList, projectId } = useProject();

const justLogged = computed(() => route.query.just_logged);
const jwtLogin = computed(() => route.query.jwt_login as string);

const { token, setToken } = useAccessToken();

const { refreshingDomains } = useDomain();
const { permission, canSeeWeb, canSeeEvents } = usePermission();

onMounted(async () => {

    if (jwtLogin.value) {
        setToken(jwtLogin.value);
        const user = await $fetch<any>('/api/user/me', { headers: { 'Authorization': 'Bearer ' + token.value } })
        const loggedUser = useLoggedUser();
        loggedUser.user = user;
    }

    if (justLogged.value) { setTimeout(() => { location.href = '/' }, 500) }

})

const firstInteraction = useFetch<boolean>('/api/project/first_interaction', {
    lazy: true, headers: useComputedHeaders({ useSnapshotDates: false })
});

const showDashboard = computed(() => project.value && firstInteraction.data.value);

const selfhosted = useSelfhosted();

</script>

<template>

    <div v-if="!canSeeWeb" class="h-full w-full flex mt-[20vh] justify-center">
        <div> You need webAnalytics permission to view this page </div>
    </div>

    <div v-if="canSeeWeb && refreshingDomains">
        <div class="w-full flex justify-center items-center mt-[20vh]">
            <i class="fas fa-spinner text-[2rem] text-accent animate-[spin_1s_linear_infinite] duration-500"></i>
        </div>
    </div>

    <div v-if="canSeeWeb && !refreshingDomains" class="dashboard w-full h-full overflow-y-auto overflow-x-hidden pb-[7rem] md:pt-4 lg:pt-0">

        <div v-if="showDashboard">

            <div class="w-full px-4 py-2 gap-2 flex flex-col">
                <BannerLimitsInfo v-if="!selfhosted" :key="refreshKey"></BannerLimitsInfo>
            </div>

            <div>
                <DashboardTopSection :key="refreshKey"></DashboardTopSection>
                <DashboardTopCards :key="refreshKey"></DashboardTopCards>
            </div>


            <div class="mt-6 px-6 flex gap-6 flex-col 2xl:flex-row w-full">
                <DashboardActionableChart v-if="canSeeWeb && canSeeEvents" :key="refreshKey"></DashboardActionableChart>
                <LyxUiCard v-else class="flex justify-center w-full py-4">
                    You need events permission to view this widget
                </LyxUiCard>
            </div>


            <div class="flex w-full justify-center mt-6 px-6">
                <div class="flex w-full gap-6 flex-col xl:flex-row">
                    <div class="flex-1">
                        <BarCardReferrers :key="refreshKey"></BarCardReferrers>
                    </div>
                    <div class="flex-1">
                        <BarCardPages :key="refreshKey"></BarCardPages>
                    </div>
                </div>
            </div>

            <div class="flex w-full justify-center mt-6 px-6">
                <div class="flex w-full gap-6 flex-col xl:flex-row">
                    <div class="flex-1">
                        <BarCardGeolocations :key="refreshKey"></BarCardGeolocations>
                    </div>
                    <div class="flex-1">
                        <BarCardDevices :key="refreshKey"></BarCardDevices>
                    </div>
                </div>
            </div>


            <div class="flex w-full justify-center mt-6 px-6">
                <div class="flex w-full gap-6 flex-col xl:flex-row">
                    <div class="flex-1">
                        <BarCardBrowsers :key="refreshKey"></BarCardBrowsers>
                    </div>
                    <div class="flex-1">
                        <BarCardOperatingSystems :key="refreshKey"></BarCardOperatingSystems>
                    </div>
                </div>
            </div>

        </div>

        <FirstInteraction v-if="!justLogged" :refresh-interaction="firstInteraction.refresh"
            :first-interaction="(firstInteraction.data.value || false)"></FirstInteraction>

        <div class="text-text/85 mt-8 ml-8 poppis text-[1.2rem]"
            v-if="projectList && projectList.length == 0 && !justLogged">
            Create your first project...
        </div>

        <div v-if="justLogged" class="text-[2rem] w-full h-full flex items-center justify-center">
            <div
                class="backdrop-blur-[1px] z-[20] left-0 top-0 w-full h-full flex items-center justify-center font-bold rockmann absolute">
                <i class="fas fa-spinner text-[2rem] text-[#727272] animate-[spin_1s_linear_infinite] duration-500"></i>
            </div>
        </div>

    </div>

</template>

<style scoped lang="scss"></style>
