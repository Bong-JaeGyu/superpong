<template>
    <HeaderNavigate />
    <ControlSidebar />

    <div class="content-wrapper dashboard">
        <ContainerHeader activeTab="dashboard_01" :regionGrpCdList="regionGrpCdList" :weatherReportList="weatherReportList"
            v-model="regionGrpCdValue" />

        <!-- 대시보드 메인 그리드 시작 -->
        <Draggable v-if="activeGridBoxList != null" v-model="activeGridBoxList" class="grid_wrap"
            ghost-class="portlet-placeholder" item-key="component" handle=".tit" forcePlaceholderSize="true"
            placeholder="sort-highlight" cancel=".content" animation="600" group="shared" @start="dragging = true"
            @end="sortDragEndEvent" @add="draggableAddEvent($event)">

            <template #item="{ element }">
                <div :class="element.class" v-show="element.visible.value">
                    <component :is="element.component" @gridEvent="gridEvent($event, element)"
                        @draggableSide="draggableSide" :ssrTypeCdList="ssrTypeCdList" :element="element"
                        :data="element.data"  @draggableEventHandler="draggableEventHandler" :cctvEventItem="cctvEventItem"/>
                </div>
            </template>

        </Draggable>

        <div v-else class="grid_wrap">
            <div class="content justify-content-center" />
        </div>
        <!-- 대시보드 메인 그리드 종료 -->

        <!-- 대시보드 최소화 그리드 시작 -->
        <div class="grid_box_menu_wrap">
            <div class="close_grid_box active" title="사이드 박스 표시"></div>

            <Draggable v-if="passiveGridBoxList != null" v-model="passiveGridBoxList" class=""
                ghost-class="portlet-placeholder" item-key="component" forcePlaceholderSize="true"
                placeholder="sort-highlight" cancel=".content" animation="600" group="shared" @start="dragging = true"
                @end="sortDragEndEvent">

                <template #item="{ element }">
                    <div :class="element.class" v-show="element.visible.value">
                        <component :is="element.component" @draggableSide="draggableSide" :ssrTypeCdList="ssrTypeCdList"
                            :element="element" :data="element.data" />
                    </div>
                </template>
            </Draggable>
        </div>
        <!-- 대시보드 최소화 그리드 종료 -->
    </div>

    <!-- 모달 컴포넌트 시작 -->
    <MeasureCurntModal ref="measureCurntModalRef" :data="measureCurntModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" :ssrTypeCdList="ssrTypeCdList" />
    <BroadcastStatusResult ref="viewResultModalRef" />

    <BroadcastQueueDetail ref="brctStartModalRef" />
        
    <MeasureRainModal ref="measureRainModalRef" :data="measureRainModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureWaterModal ref="measureWaterModalRef" :data="measureWaterModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureGWTModal ref="measureGWTModalRef" :data="measureGWTModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureSDModal ref="measureSDModalRef" :data="measureSDModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureSSModal ref="measureSSModalRef" :data="measureSSModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureEQModal ref="measureEQModalRef" :data="measureEQModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureEQERModal ref="measureEQERModalRef" :data="measureEQERModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureWVModal ref="measureWVModalRef" :data="measureWVModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />
    <MeasureICModal ref="measureICModalRef" :data="measureICModalData" :regionGrpCd="regionGrpCdRef"
        @measureCurntModalEvent="measureCurntModalEvent" />

    <!-- 모달 컴포넌트 종료 -->

    <div class="d-none">
        <!-- 피드백을 방지하기 위해 항상 음소거되어야 함 -->
        <video id="localVideo" muted="muted" width="1"></video>
        <!-- 호출되는 사람의 비디오를 표시하는  remoteVideo -->
        <video id="remoteVideo"></video>
    </div>

    <Loading2 :visible="loadingVisible" />
</template>

<script setup>
import { initLogger } from '@/common/logger';
const logger = initLogger('@/broadcast-schdule/editor', 'debug');

import { createPermissionMap } from '@/common/permissionMap';

/** ################################### 컴포넌트 시작 */
/** Draggable 컴포넌트 */
import Draggable from "vuedraggable";

/** 사이드 메뉴 */
import ControlSidebar from './../../../components/sidemenu/ControlSidebar.vue';
/** 관리지역, 탭, 기상특보 정보가 있는 헤더 */
import ContainerHeader from './../../../components/header/ContainerHeader.vue';
/** 헤더 (다크모드가 있는 대시보드/지도서비스용 헤더) */
import HeaderNavigate from '../map-service/header/HeaderNavigate.vue';

import BroadcastStatusResult from './../../broadcast/components/BroadcastStatusResult';     // 방송결과조회
/** ################################### 컴포넌트 종료 */

/** ################################### 모달 컴포넌트 시작 */
import MeasureCurntModal from './../modal/MeasureCurntModal';
// import ViewResultModal from './../modal/ViewResultModal';
// import BrctStartModal from './../modal/BrctStartModal';
import BroadcastQueueDetail from './../../broadcast/components/BroadcastQueueDetail.vue';

import MeasureRainModal from './../modal/MeasureRainModal';
import MeasureWaterModal from './../modal/MeasureWaterModal';
import MeasureGWTModal from './../modal/MeasureGWTModal';
import MeasureSDModal from './../modal/MeasureSDModal';
import MeasureSSModal from './../modal/MeasureSSModal';
import MeasureEQModal from './../modal/MeasureEQModal';
import MeasureEQERModal from './../modal/MeasureEQERModal';
import MeasureWVModal from './../modal/MeasureWVModal';
import MeasureICModal from './../modal/MeasureICModal';

/** ################################### 모달 컴포넌트 종료 */

/** ################################### 바인딩 데이터 시작 */

/** 대시보드 그리드 관련 객체들을 생성해주는 함수 */
import { createGridBoxFn } from './gridPannel/controller/common';
/** 관리 지역 그룹 리스트 관련 객체들을 생성해주는 함수 */
import { createRegionGrpCdList } from './../datas/regionGrpCdList';
/** 대시보드와 그리드간 데이터 공유시 사용하는 Context */
import { createDashboardContext } from './../datas/dashboardContext';
/** 계측 현황 모달 핸들 */
import { createMeasureCurrentModalHandle } from './../modal/measureCurrent/control';
import { onBeforeUnmount, onMounted, ref } from 'vue';

import {
    CHANNEL_DASHBOARD, CHANNEL_BROADCAST, CHANNEL_MODULE, SocketConnection,
    ROUTING_RESPONSE_WARN_LOG,
    ROUTING_RESPONSE_AIR_LOG,
    ROUTING_RESPONSE_FCST_LOG,
    ROUTING_RESPONSE_SRT_FSCT_LOG,
    // ROUTING_TMN_REQ_BCAST_LOG,
    ROUTING_TMN_RES_BCAST_LOG,
    // ROUTING_CALL_REQ_BCAST_CONN_LOG,
    ROUTING_CALL_RES_BCAST_CONN_LOG,
    ROUTING_SSR_RES_MEASURE_LOG,
    ROUTING_SSR_RES_MEASURE_DISP_LOG,
    ROUTING_TMN_RES_VOIP_INSP_LOG,
    ROUTING_WEATHER_RES_WARN_LOG,
    ROUTING_WEATHER_RES_AIR_LOG,
    ROUTING_WEATHER_RES_FCST_LOG,
    ROUTING_WEATHER_RES_SRT_FSCT_LOG,
    ROUTING_WEATHER_RES_SEA_FSCT_LOG

} from "./../../../websocket/SocketConnection";

// import { useStore } from "vuex";
import { mainApi } from './../../../api/dashboard';

import { useRouter } from "vue-router";
import { isEmpty } from "./../../../common/utils";
/** ################################### 바인딩 데이터 종료 */

const loadingVisible = ref(false);

const {
    /** 관리 지역 그룹 리스트 목록 */
    regionGrpCdList,
    /** 관리 지역 그룹 선택된 값 */
    regionGrpCdValue,
    /** 관리 지역 그룹 선택된 값 변경 감지 */
    regionGrpCdWatchCallback,
    /** REST API 로 관리지역 그룹코드 목록을 갱신합니다 */
    updateRegionGrpCdAndSsrTypeCdList,
    /** 관리 지역 그룹 리스트 관련 객체들을 생성해주는 함수 */
} = createRegionGrpCdList();

const {
    /** 선택된 관리 지역코드 (setRegionGrpCd 로 설정해야 합니다) */
    regionGrpCdRef,
    /** 그리드와 모달간 데이터 공유에 사용할 Context */
    dashboardContext,
    /** 관리지역 코드를 설정합니다 */
    setRegionGrpCd,
    /** 모달을 참조할 Ref 을 반환 합니다 */
    getModalRef,
    /** 모달에 대한 Reactive Value 를 반환합니다 */
    getModalData,
    /** 참조가 생성된 모달들을 닫습니다 */
    closeModals,

    getInstituteCode
    /** 대시보드와 그리드간 데이터 공유시 Context 를 생성해주는 함수 */
} = createDashboardContext({ loadingVisible, regionGrpCdList });

const {
    /** 계측현황 모달 참조 */
    measureCurntModalRef,
    /** 계측현황 모달 Reactive Data */
    measureCurntModalData,
    /** 계측현황 모달 이벤트 핸들 */
    measureCurntModalEvent
    /**계측현황 */
} = createMeasureCurrentModalHandle(dashboardContext);

/** 그리드 이벤트 핸들 함수 */    
let cctvEventItem = ref('')
const draggableEventHandler = (args = {}) => {
    const type = args.type;
    
    if (type === 'measureCctvEvent') {
        const dvcLat = args.item.dvcLat;
        const dvcLon = args.item.dvcLon;

        if (!isEmpty(dvcLat) && !isEmpty(dvcLon)) {
            cctvEventItem.value = args.item;
        } else {
            alert('좌표 위치를 찾을 수 없습니다.')
        }

    }
};

const viewResultModalRef = getModalRef('viewResultModal');
const brctStartModalRef = getModalRef('brctStartModal');

//RN 강우
const measureRainModalData = getModalData('measureRainModal');
const measureRainModalRef = getModalRef('measureRainModal');

//WT 수위
const measureWaterModalData = getModalData('measureWaterModal');
const measureWaterModalRef = getModalRef('measureWaterModal');

//GWT 지하수위
const measureGWTModalData = getModalData('measureGWTModal');
const measureGWTModalRef = getModalRef('measureGWTModal');

//SD 지표변위
const measureSDModalData = getModalData('measureSDModal');
const measureSDModalRef = getModalRef('measureSDModal');

//SS 구조물경사
const measureSSModalData = getModalData('measureSSModal');
const measureSSModalRef = getModalRef('measureSSModal');

//EQ 지진
const measureEQModalData = getModalData('measureEQModal');
const measureEQModalRef = getModalRef('measureEQModal');

//EQER 지진조기경보
const measureEQERModalData = getModalData('measureEQERModal');
const measureEQERModalRef = getModalRef('measureEQERModal');

//WV 유속
const measureWVModalData = getModalData('measureWVModal');
const measureWVModalRef = getModalRef('measureWVModal');

//IC 경사계
const measureICModalData = getModalData('measureICModal');
const measureICModalRef = getModalRef('measureICModal');

/** 관리지역 리스트 조회(갱신) */
updateRegionGrpCdAndSsrTypeCdList();

/** 기상 특보 목록 */
const weatherReportList = ref([]);
/** 기상 특보 목록을 조회 */
async function updateWeatherReportList() {
    const data = await mainApi({
        "procedureIdList": "WG_SPC_WEATHER",
        "inputParameters": {
            "WG_SPC_WEATHER": {
                regionGrpCd: dashboardContext.regionGrpCd
            }
        }
    });

    const result = data.body.WG_SPC_WEATHER;
    if (result.outputParameters.result < 0) {
        return;
    }

    if (result.items != null && Array.isArray(result.items)) {

        try {
            result.items.forEach(item => {
                item.className = item.warnStress == 0 ? 'lv04' : 'lv05';
            });

            weatherReportList.value = result.items;
        } catch (reason) {
            logger(reason);
        }
    } else {
        weatherReportList.value = [];
    }

}
// updateWeatherReportList();

const {
    activeGridBoxList,
    passiveGridBoxList,
    sortDragEndEvent,
    draggableSide,
    draggableAddEvent,
    updateGridData,
    updatePannelData,
    updateGridVisible,
    gridEvent
} = createGridBoxFn(dashboardContext);

/** 관리지역 변경시 보이는 패널 종류와 데이터 갱신 */
regionGrpCdWatchCallback(async function (regionGrpCdValue) {
    setRegionGrpCd(regionGrpCdValue);

    loadingVisible.value = true;

    try {

        await updateGridVisible();
    } catch (reason) {
        logger(reason);
    } finally {
        // loadingVisible.value = false;
    }
    try {
        // loadingVisible.value = true;
        await updateGridData();
    } catch (reason) {
        logger(reason);
    } finally {
        // loadingVisible.value = false;
    }
    try {
        // loadingVisible.value = true;
        await updateWeatherReportList();
    } catch (reason) {
        logger(reason);
    } finally {
        // loadingVisible.value = false;
    }
    loadingVisible.value = false;
});

onBeforeUnmount(() => {
    closeModals();
});

window.socketConnection.useEventListener(CHANNEL_DASHBOARD, SocketConnection.EVENT_MESSAGE, async function (message) {
    try {
        logger('socketConnection', CHANNEL_DASHBOARD, event.jsonData);
        loadingVisible.value = true;

        switch (message.jsonData.header.routingKey) {

            case ROUTING_CALL_RES_BCAST_CONN_LOG: //Call Daemon - MQTT 프로토콜 방송 연결응답 safety.call.res.bcast-conn.log
                updatePannelData(['WG_BRCT_TODAY', 'WG_BRCT_RECENT', 'WG_TXMR', 'WG_MAP']); //오늘방송, 최근방송, 방송단말, 지도
                break;

            case ROUTING_RESPONSE_AIR_LOG:    //2.8.대기오염 대기 정보 수집 response.air.log
            case ROUTING_WEATHER_RES_AIR_LOG: //대기오염 대기 정보 수집 safety.weather.res.air.log
                updatePannelData(['WG_AIR']); //대기정보
                break;

            case ROUTING_SSR_RES_MEASURE_LOG:                       //계측데이터 저장 (마을방송) safety.ssr.res.measure.log
                updatePannelData(['WG_SSR_LVL_VILG', 'WG_MAP']);    //마을방송 계측데이터
                break;

            case ROUTING_SSR_RES_MEASURE_DISP_LOG:                  //계측데이터 저장 (재해위험지구) safety.ssr.res.measure-dist.log
                updatePannelData(['WG_SSR_LVL_DIST', 'WG_MAP']);    //재해위험 계측데이터
                break;

            case ROUTING_RESPONSE_FCST_LOG:         //[클라우드] 2.9.단기예보 - 날씨 정보 response.fcst.log
            case ROUTING_WEATHER_RES_FCST_LOG:      //[기상DMN] 단기예보 - safety.weather.res.fcst.log
            case ROUTING_RESPONSE_SRT_FSCT_LOG:     //[클라우드] 2.12.초단기실황 - 날씨 정보 response.srtfsct.log
            case ROUTING_WEATHER_RES_SRT_FSCT_LOG:  //[기상DMN] 초단기실황 - 날씨 정보 safety.weather.res.srtfcst.log
                updatePannelData(['WG_WEATHER']);   //날씨정보A, 날씨정보B, 날씨정보C
                break;

            case ROUTING_RESPONSE_WARN_LOG:     //기상 특보 response.warn.log
            case ROUTING_WEATHER_RES_WARN_LOG:  // 기상 특보 safety.weather.res.warn.log
                updateWeatherReportList();
                break;
        }
    } catch (reason) {
        logger(reason);
    } finally {
        loadingVisible.value = false;
    }
});

</script>

<!-- 대시보드 / 지도서비스에 적용할 CSS -->
<style src="./../../../scopeStyles/dashboard.css"></style>